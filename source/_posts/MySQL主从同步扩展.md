---
title: MySQL主从同步扩展
author: Evobot
date: 2018-07-01 00:31:01
categories: MySQL
tags:
  - MySQL
  - Centos
image:
---



本篇文章主要介绍如何使用xtrabackup实现不停库、不锁表配置主从复制，另外介绍如何实现MySQL主主复制，以及在主主基础上实现环形复制，最后介绍了使用mysql-proxy实现读写分离配置。

<!--more-->

---

# 不停库锁表配置主从

## 主从复制类型

- 前面介绍过xtrabackup软件用来对MySQL进行备份，而在配置MySQL主从时，我们由于需要备份主服务器的数据库，所以需要停库使用mysqldump备份，另外在配置主服务器时，还进行锁表以便查看File和Position，这里我们使用xtrabackup帮助我们进行不停库锁表配置主从同步；
- MySQL主从复制分为三种类型：
  - 基于语句的复制：`STATEMENT`，在主服务器上执行SQL语句，在从服务器上执行同样的语句，有可能会由于SQL执行上下文环境不同而使数据不一致，MySQL5.7.7以前采用基于语句复制，之后默认采用`row-based`；
  - 基于行的复制：`ROW`，把改变的内容复制过去，而不是把命令在从服务器上执行一遍，从MySQL5.0开始支持，能严格保证数据完全一致，但使用`mysqlbinlog`分析日志变得没有意义，因为任何一条update语句，会将涉及到的行数据全部set值，所以binlog文件比较大；
  - 混合类型的复制：`MIXED`，默认采用基于语句的复制，一旦发现基于语句的复制无法精确复制时，就会采用基于行的复制。
- 在MySQL命令行中，可以使用`SET SESSION binlog_format=MIXED;`命令来修改二进制日志的类型，需要super权限；

## 使用xtrabackup不停库配置主从

1. 首先与上一篇文章[MySQL主从配置](https://www.evobot.cn/post/f1a9d93b.html) 相同，但是不进行锁表操作，也不用使用`show master status`查看File和Position；

2. 在备份数据库的步骤上，不再采用mysqldump，而是使用XtraBackup对主库进行全量备份，步骤与之前的文章[MySQL 用户管理、常用 SQL 语句及数据库备份与恢复](https://www.evobot.cn/post/c54c87e0.html)相同；

3. 将备份文件打包或使用rsync传输到从服务器上并进行恢复操作，在执行恢复操作时，执行`innobackupex --use-memory=16G --apply-log 2018-06-30_00-00-02`时，会在输出中显示主库的FILE和Position，记录下来备用：

   ```bash
   $ innobackupex --apply-log /data/backup/2018-07-01_02-00-16/
   .....
   InnoDB: xtrabackup: Last MySQL binlog file position 56460317, file name evobotlinux1.000002

   xtrabackup: starting shutdown with innodb_fast_shutdown = 1
   ....
   ```

4. 恢复完成后，配置从库的my.cnf，并且在从库my.cnf的mysqld配置块中加入`read_only = 1`将从库的写入权限关闭，或者在MySQL命令行中执行`set global read_only=1;`关闭；

5. 之后的从库配置与之前的[MySQL主从配置](https://www.evobot.cn/post/f1a9d93b.html) 从库配置相同，最后使用`start slave`启动从库即可。


- 本节参考博文[使用 Xtrabackup 在线对MySQL做主从复制](http://seanlook.com/2015/12/14/mysql-replicas/)。

---

# MySQL主主复制

## MySQL主主复制介绍

- 主主复制是指在主从复制的基础上，增加一台从服务器或利用原有的一台从服务器，将其设置为可读写，与原主服务器互为主备；
- 例如masterA是masterB的主库，同时masterB也是masterA的主库，互为主从；
- 互为主从的两台服务器，可能会导致masterB一直处于空闲的写状态，所以可以将其作为从库，负责部分查询；

   简易的架构图如下：

![MySQL双主复制](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/mysql-master.jpg)

## 主主复制配置

- 首先与MySQL主从复制的配置相同，配置两台MySQL服务器进行主从同步；

  主服务器的my.cnf中mysqld配置块增加以下配置：

  ```bash
  auto_increment_offset = 1
  auto_increment_increment = 2
  log-slave-updates = true
  gtid-mode = on
  enforce-gtid-consistency = true
  binlog-format = ROW
  ```

  从服务器的my.cnf的mysqld同样也增加以下配置：

  ```bash
  auto_increment_offset = 2
  auto_increment_increment = 2
  log-slave-updates = true
  gtid-mode = on
  enforce-gtid-consistency = true
  binlog-format=ROW
  ```

  ​

  - 其中auto_increment_*配置是为了防止两台主库同时对一个表进行写操作，导致自增键相同发生冲突，所以将主服务器的自增ID的值设置为1，增长为2，即第一条记录的ID为1 ，第二条记录的ID为3；
  - 而从服务器的auto_increment_offset则设置为2，表示第一条记录的ID为2，第二条则是4，从而实现两台主服务器的自增ID不发生冲突；
  - 而log-slave-updates = true则表示将复制事件写入binlog，一台服务器既做主库又做从库必须开启此选项。
  - 而最后三行是配置MySQL支持GTID复制，GTID(Global Transaction Identifiers)是全局事务标识，当使用GTIDS时，在主上提交的每一个事务都会被识别和跟踪，并且运用到所有从MySQL，而且配置主从或者主从切换时不再需要指定 master_log_files和master_log_pos；由于GTID-base复制是完全基于事务的，所以能很简单的决定主从复制的一致性，官方建议Binlog采用Row格式；

- 完成主从配置后，在从库上创建用于主从同步的用户，然后在主库上配置同步，与从库配置相同，在从库执行`show master status`获得File和Position，执行下面的命令：

  ```sql
  change master to master_host='192.168.199.129',master_user='repl',master_password='password',master_log_file='evobotlinux2.000001',master_log_pos=613;

  start slave;

  show slave status\G
  ```

  - 这里的change master语句的信息则是填写从库的IP地址和用户名密码等。

- 由于我们配置了GTID，实际上配置change master的时候，可以不用写File和Position，而使用下面的命令：

  ```sql
  CHANGE MASTER TO MASTER_HOST='192.168.199.129',MASTER_USER='repl',MASTER_PASSWORD='password',MASTER_AUTO_POSITION=1;
  ```

- 如果已经配置完成了主主复制，那么开启GTID功能只需要在从服务器上执行以下命令（这里两台主库同样也是从库，所以两台主库都需要执行）：

  ```sql
  stop slave;

  change master to MASTER_AUTO_POSITION=1;

  start slave;
  ```

- 实际上将masterB作为主之后，可以增加多个从服务器，都设置为主服务器，实现masterA->masterB->masterC->masterA环形主从复制。

- 本文参考博文[MySQL双主（主主）架构方案](https://www.cnblogs.com/ygqygq2/p/6045279.html)。

---

# mysql-proxy实现读写分离

- 下载安装mysql-proxy:

  ```bash
  $ wget https://downloads.mysql.com/archives/get/file/mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz

  $ tar zxvf ysql-proxy-0.8.5-linux-glibc2.3-x86-64bit.tar.gz

  $ mv mysql-proxy-0.8.5-linux-glibc2.3-x86-64bit /usr/local/mysql-proxy
   
  ```

- 创建配置文件：

  ```bash
  $ mkdir /usr/local/mysql-proxy/conf
  $ mkdir /usr/lcoal/mysql-proxy/log

  $ vi /usr/local/mysql-proxy/conf/mysql-proxy.conf
  ```

  写入以下内容：

  ```bash
  [mysql-proxy]
  daemon=true   #后台运行
  user=root     #mysql-proxy运行用户
  keepalive=true
  plugins=proxy,admin
  log-level=info    #日志级别
  log-file=/usr/local/mysql-proxy/log/mysql-proxy.log  ##proxy日志地址
  proxy-address=192.168.199.127:4040  #本机ip地址      
  proxy-backend-addresses=192.168.199.128:3306  ##backend主
  proxy-read-only-backend-addresses=192.168.199.129:3306  ##backend从
  proxy-lua-script=/usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua    ##lua脚本地址
  admin-address=192.168.199.127:4041   ##proxy的管理用户adminiphe端口
  admin-username=admin	# mysql-proxy管理用户
  admin-password=password # 管理用户密码
  admin-lua-script=/usr/local/mysql-proxy/lib/mysql-proxy/lua/admin.lua   #admin的lua脚本地址；
  ```

- 更改lua脚本配置，使其能够尽快进入读写分离状态：

  ```bash
  $ vi /usr/local/mysql-proxy/share/doc/mysql-proxy/rw-splitting.lua

  -- connection pool
  if not proxy.global.config.rwsplit then
          proxy.global.config.rwsplit = {
                  min_idle_connections = 1,  ##最小连接数
                  max_idle_connections = 2,  ##最大连接数后实现读写分离

                  is_debug = false
          }
  ```

- 运行mysql-proxy:

  ```bash
  $ /usr/local/mysql-proxy/bin/mysql-proxy --defaults-file=/usr/local/mysql-proxy/conf/mysql-proxy.conf
  ```

- 查看日志，已经控制主从服务器：

  ```bash
  $ cat /usr/local/mysql-proxy/log/mysql-proxy.log 
  2018-07-02 22:48:33: (message) chassis-unix-daemon.c:136: [angel] we try to keep PID=4582 alive
  2018-07-02 22:48:33: (critical) plugin proxy 0.8.5 started
  2018-07-02 22:48:33: (critical) plugin admin 0.8.5 started
  2018-07-02 22:48:33: (message) proxy listening on port 192.168.199.127:4040
  2018-07-02 22:48:33: (message) added read/write backend: 127.0.0.1:3306
  2018-07-02 22:48:33: (message) added read-only backend: 192.168.199.129:3306
  2018-07-02 22:48:34: (message) admin-server listening on port 192.168.199.127:4041
  2018-07-02 23:19:34: (message) Initiating shutdown, requested from signal handler
  2018-07-02 23:19:34: (message) shutting down normally, exit code is: 0

  ```

- 然后在**主从服务器**上给mysql-proxy创建登录用户。如果配置了主从同步，那么只需要在主服务器上创建即可，从服务器会自动同步：

  ```sql
  grant all on *.* to 'root'@'%' identified by '123456';
  ```

- 访问mysql-proxy：

  ```bash
  $ mysql -h129.168.199.127 -P4040 -uevobot -p   ##通过访问proxy，直接转到mysql
  ```

- 使用上面的命令多次连接mysql-proxy，当连接数大于2(lua脚本中的配置)时，就会启动读写分离机制：

  ```bash
  $ mysql -h192.168.199.127 -u admin -ppassword -P 4041;
  Welcome to the MariaDB monitor.  Commands end with ; or \g.
  Your MySQL connection id is 1
  Server version: 5.0.99-agent-admin

  Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

  MySQL [(none)]> select * from backends;
  +-------------+------------------+---------+------+------+-------------------+
  | backend_ndx | address          | state   | type | uuid | connected_clients |
  +-------------+------------------+---------+------+------+-------------------+
  |           1 | 192.168.199.128:3306 | up      | rw   | NULL |                 0 |
  |           2 | 192.168.199.129:3306 | unknown | ro   | NULL |                 0 |
  +-------------+------------------+---------+------+------+-------------------+
  2 rows in set (0.00 sec)
  ```

- 由于MySQL官方不推荐将mysql-proxy用于生产环境，所以实现读写分离，还可以尝试使用mycat和基于mysql-proxy二次开发的atlas。

---



