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

- 本文参考博文[MySQL双主（主主）架构方案](https://www.cnblogs.com/ygqygq2/p/6045279.html)。

---



