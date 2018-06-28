---
title: MySQL主从配置
author: Evobot
date: 2018-06-28 22:30:11
categories: Mysql
tags: 
  - Mysql
  - Centos
image:
---



本文主要介绍MySQL主从同步的概念，然后介绍了如何配置主从同步，并对主从同步进行了测试。

<!--more-->

---

# MySQL主从介绍

- MySQL主从又叫做Replication、AB复制，即A、B两台机器做主从后，在A上写数据，B上也会跟着写数据，两者数据实时同步；

- MySQL主从是基于binlog的，主上必须开启binlog才能进行主从；

- 主从过程分为以下三个步骤：

  - 主将更改操作记录到binlog内；
  - 从将主的binlog事件(SQL语句)同步到本机上并记录在relaylog里，也叫中继日志；
  - 从根据relaylog里面的SQL语句按顺序执行；

- 主上有一个log dump线程，用来和从的I/O线程传递binlog；

- 从上有两个线程，其中I/O线程用来同步主的binlog并生成relaylog，另外一个SQL线程用来把relaylog里的SQL语句落地执行；

- MySQL主从原理如下图：

  ![mysql-master](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/MySQL-master.png)

![mysql-master](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/MySQL-mater2.png)

-  主从同步的作用首先是为了数据的备份，其次也可以实现MySQL的从库读取，减轻主的压力。

# MySQL主机配置

- 修改my.cnf，在`[mysqld]`块中增加server_id和log_bin配置：

  ```bash
  [mysqld]
   basedir = /usr/local/mysql
   datadir = /data/mysql
   port = 3306
   server_id = 128
   log_bin = evobotlinux1
   socket = /tmp/mysql.sock
  ```

  - server_id和log_bin的配置都是自定义的值。

- 然后重启MySQL，查看MySQL的datadir目录，会生成以log_bin命名的文件：

  ```bash
  $ cd /data/mysql
  $ ls
  auto.cnf             ib_logfile1                sys
  employees            ibtmp1                     testdb
  evobotlinux1.000001  localhost.localdomain.err  testdb2
  evobotlinux1.index   localhost.localdomain.pid  xtrabackup_info
  ib_buffer_pool       mysql                      xtrabackup_master_key_id
  ibdata1              mysqld_safe.pid            zrlog
  ib_logfile0          performance_schema

  ```

  - 可以看到生成了evobotlinux1为名的文件，这几个文件是实现主从配置的根本文件。

- 接下来对数据库进行一些操作，如重新创建一个数据库：

  ```bash
  $ /usr/local/mysql/bin/mysqldump -uroot -p zrlog > /tmp/blog.sql
  Enter password: 

  $ mysql -uroot -p -e "create database blog"
  Enter password: 

  $ mysql -uroot -p blog < /tmp/blog.sql 

  ```

- 在MySQL中创建用于主从同步的用户：

  ```sql
  grant replication slave on *.* to 'repl'@'192.168.199.129' identified by 'password';
  ```

  > 这里只给予了replication和slave权限用于主从同步，并且新建的用户repl只允许从机器192.168.199.129上登录。

- 接着进行锁表操作，使数据库不再进行写操作：

  ```sql
  flush tables with read lock;
  ```

- 查看主的状态：

  ```sql
  mysql> show master status;
  +---------------------+----------+--------------+------------------+-------------------+
  | File                | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
  +---------------------+----------+--------------+------------------+-------------------+
  | evobotlinux1.000001 |    33803 |              |                  |                   |
  +---------------------+----------+--------------+------------------+-------------------+
  1 row in set (0.00 sec)

  ```

  > 需要主的File和Position列的值。

- 然后退出MySQL命令行，并将需要主从同步的数据库进行备份，其中名为mysql的数据库可以不进行备份，因为这个数据库存储的是用户名、权限等信息，不需要进行备份：

  ```bash
  $ mysqldump -uroot -p123456 employees > /tmp/employees.sql
  ```

# MySQL从机配置

- 修改my.cnf，将server_id定义为与主不同的值，不需要配置log_bin:

  ```bash
  [mysqld]
   basedir = /usr/local/mysql
   datadir = /data/mysql
   port = 3306
   server_id = 129
   socket = /tmp/mysql.sock

  ```

- 重启MySQL服务，从主上将备份的数据库文件复制到从上来：

  ```bash
  $ scp 192.168.199.128:/tmp/*.sql /tmp/
  root@192.168.199.128's password: 
  blog.sql                                                    100%   31KB  31.2KB/s   00:00    
  employees.sql                                               100%  161MB  80.3MB/s   00:02   
  ```

- 在MySQL中创建需要主从同步的数据库：

  ```sql
  mysql> create database employees;
  Query OK, 1 row affected (0.00 sec)

  mysql> create database zrlog;
  Query OK, 1 row affected (0.00 sec)

  mysql> create database blog;
  Query OK, 1 row affected (0.00 sec)

  ```

- 然后把主上复制过来的数据库备份恢复到对应的数据库中：

  ```bash
  $ mysql -uroot -p zrlog < /tmp/blog.sql 
   
  $ mysql -uroot -p blog < /tmp/blog.sql 

  $ mysql -uroot -p employees < /tmp/employees.sql 

  ```

- 接着进行主从同步配置，登录MySQL，执行以下SQL语句：

  ```sql
  stop slave;
  ```

  ```sql
  change master to master_host='192.168.199.128',master_port=3306,master_user='repl',master_password='password',master_log_file='evobotlinux1.000001',master_log_pos=33803;
  ```

  - 执行主的ip，port，用户名、密码，而`master_log_file`和`master_log_pos`就是先前在主上的MySQL中，执行`show master status`显示的信息。

- 执行`start slave;`开启主从，判断主从是否正常，需要执行`show slave status\G`命名，在输出中，如果`Slave_IO_Running`和`Slave_SQL_Running`两个项都是YES，则主从同步正常，只要存在一个NO，则表示主从断开，需要排除错误;

- 另外还需要关注`Seconds_Behind_Master`主从延迟时间信息、`Last_IO_Errno`IO错误数量、`Last_IO_Error`IO错误信息、`Last_SQL_Errno`SQL错误数量和`Last_SQL_Error`SQL错误信息；

  **错误主从同步信息：**

  如果出现错误，那么可以重启主从查看是否恢复，无法恢复时，需要在从上重新执行change_master语句。

  ```bash
  mysql> show slave status\G
  *************************** 1. row ***************************
                 Slave_IO_State: Waiting for master to send event
                    Master_Host: 192.168.199.128
                    Master_User: repl
                    Master_Port: 3306
                  Connect_Retry: 60
                Master_Log_File: evobotlinux1.000001
            Read_Master_Log_Pos: 34122
                 Relay_Log_File: localhost-relay-bin.000002
                  Relay_Log_Pos: 486
          Relay_Master_Log_File: evobotlinux1.000001
               Slave_IO_Running: Yes
              Slave_SQL_Running: No
                Replicate_Do_DB: 
            Replicate_Ignore_DB: 
             Replicate_Do_Table: 
         Replicate_Ignore_Table: 
        Replicate_Wild_Do_Table: 
    Replicate_Wild_Ignore_Table: 
                     Last_Errno: 1008
                     Last_Error: Error 'Can't drop database 'testdb2'; database doesn't exist' on query. Default database: 'testdb2'. Query: 'drop database testdb2'
                   Skip_Counter: 0
            Exec_Master_Log_Pos: 33966
                Relay_Log_Space: 1233
                Until_Condition: None
                 Until_Log_File: 
                  Until_Log_Pos: 0
             Master_SSL_Allowed: No
             Master_SSL_CA_File: 
             Master_SSL_CA_Path: 
                Master_SSL_Cert: 
              Master_SSL_Cipher: 
                 Master_SSL_Key: 
          Seconds_Behind_Master: NULL
  Master_SSL_Verify_Server_Cert: No
                  Last_IO_Errno: 0
                  Last_IO_Error: 
                 Last_SQL_Errno: 1008
                 Last_SQL_Error: Error 'Can't drop database 'testdb2'; database doesn't exist' on query. Default database: 'testdb2'. Query: 'drop database testdb2'
    Replicate_Ignore_Server_Ids: 
               Master_Server_Id: 128
                    Master_UUID: 9bbc8648-7894-11e8-a685-000c2982ee84
               Master_Info_File: /data/mysql/master.info
                      SQL_Delay: 0
            SQL_Remaining_Delay: NULL
        Slave_SQL_Running_State: 
             Master_Retry_Count: 86400
                    Master_Bind: 
        Last_IO_Error_Timestamp: 
       Last_SQL_Error_Timestamp: 180629 01:00:31
                 Master_SSL_Crl: 
             Master_SSL_Crlpath: 
             Retrieved_Gtid_Set: 
              Executed_Gtid_Set: 
                  Auto_Position: 0
           Replicate_Rewrite_DB: 
                   Channel_Name: 
             Master_TLS_Version: 
  1 row in set (0.00 sec)

  ```

  **正确主从同步信息:**

  ```bash
  mysql> show slave status\G
  *************************** 1. row ***************************
                 Slave_IO_State: Waiting for master to send event
                    Master_Host: 192.168.199.128
                    Master_User: repl
                    Master_Port: 3306
                  Connect_Retry: 60
                Master_Log_File: evobotlinux1.000001
            Read_Master_Log_Pos: 34122
                 Relay_Log_File: localhost-relay-bin.000004
                  Relay_Log_Pos: 323
          Relay_Master_Log_File: evobotlinux1.000001
               Slave_IO_Running: Yes
              Slave_SQL_Running: Yes
                Replicate_Do_DB: 
            Replicate_Ignore_DB: 
             Replicate_Do_Table: 
         Replicate_Ignore_Table: 
        Replicate_Wild_Do_Table: 
    Replicate_Wild_Ignore_Table: 
                     Last_Errno: 0
                     Last_Error: 
                   Skip_Counter: 0
            Exec_Master_Log_Pos: 34122
                Relay_Log_Space: 703
                Until_Condition: None
                 Until_Log_File: 
                  Until_Log_Pos: 0
             Master_SSL_Allowed: No
             Master_SSL_CA_File: 
             Master_SSL_CA_Path: 
                Master_SSL_Cert: 
              Master_SSL_Cipher: 
                 Master_SSL_Key: 
          Seconds_Behind_Master: 0
  Master_SSL_Verify_Server_Cert: No
                  Last_IO_Errno: 0
                  Last_IO_Error: 
                 Last_SQL_Errno: 0
                 Last_SQL_Error: 
    Replicate_Ignore_Server_Ids: 
               Master_Server_Id: 128
                    Master_UUID: 9bbc8648-7894-11e8-a685-000c2982ee84
               Master_Info_File: /data/mysql/master.info
                      SQL_Delay: 0
            SQL_Remaining_Delay: NULL
        Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
             Master_Retry_Count: 86400
                    Master_Bind: 
        Last_IO_Error_Timestamp: 
       Last_SQL_Error_Timestamp: 
                 Master_SSL_Crl: 
             Master_SSL_Crlpath: 
             Retrieved_Gtid_Set: 
              Executed_Gtid_Set: 
                  Auto_Position: 0
           Replicate_Rewrite_DB: 
                   Channel_Name: 
             Master_TLS_Version: 
  1 row in set (0.00 sec)

  ```

- 最后，在主上的MySQL中，执行`unlock tables;`解除锁定写操作。

#  测试主从同步

## 主从补充配置

- 补充一些主从上的配置，下面的配置都是需要在my.cnf中进行配置的：

  **主服务器上：**

  - `binlog-do-db=`	表示仅同步指定的库，多个库使用英文逗号分隔
  - `binlog-ignore-db=`    表示忽略同步指定库

  **从服务器上：**

  - `replicate_do_db`   仅同步指定库
  - `replicate_ignore_db=` 忽略同步指定库
  - `replicate_do_table=`  同步指定的表
  - `replicate_ignore_table=`  忽略同步指定的表

  > 由于上面的do_table和ignore_table会导致使用use somedb语句时，执行了类似`select * from db.tb1`语句时，会导致不记录SQL语句，所以使用下面的形式代替上面的忽略表配置。

  - `replication_wild_do_table=`   支持通配符，如`evobot.%`，所以也可以用来忽略库。
  - `replication_wild_ignore_table=` 忽略指定表，如`evobot.%`

## 主从测试

**主服务器操作：**

- 在主服务器查看数据库表的记录数：

  ```sql
  mysql> use employees;

  mysql> show tables;
  +----------------------+
  | Tables_in_employees  |
  +----------------------+
  | current_dept_emp     |
  | departments          |
  | dept_emp             |
  | dept_emp_latest_date |
  | dept_manager         |
  | employees            |
  | salaries             |
  | titles               |
  +----------------------+
  8 rows in set (0.00 sec)

  mysql> select count(*) from titles;
  +----------+
  | count(*) |
  +----------+
  |   443308 |
  +----------+
  1 row in set (0.12 sec)

  ```

- 清空上面的titles表：

  ```sql
  mysql> truncate titles;
  Query OK, 0 rows affected (0.18 sec)

  mysql> select count(*) from titles;
  +----------+
  | count(*) |
  +----------+
  |        0 |
  +----------+
  1 row in set (0.00 sec)

  ```

**从服务器查看表**

- 从服务器同样查看titles表：

  ```sql
  mysql> select count(*) from titles;
  +----------+
  | count(*) |
  +----------+
  |        0 |
  +----------+
  1 row in set (0.00 sec)

  ```

  - 从服务器上的数据也被清空。

**主服务器操作**

- 删除titles表：

  ```sql
  mysql> drop table titles;
  Query OK, 0 rows affected (0.16 sec)

  mysql> show tables;
  +----------------------+
  | Tables_in_employees  |
  +----------------------+
  | current_dept_emp     |
  | departments          |
  | dept_emp             |
  | dept_emp_latest_date |
  | dept_manager         |
  | employees            |
  | salaries             |
  +----------------------+
  7 rows in set (0.00 sec)

  ```

**从服务器**

- 查看titles表是否还在：

  ```sql
  mysql> show tables;
  +----------------------+
  | Tables_in_employees  |
  +----------------------+
  | current_dept_emp     |
  | departments          |
  | dept_emp             |
  | dept_emp_latest_date |
  | dept_manager         |
  | employees            |
  | salaries             |
  +----------------------+
  7 rows in set (0.00 sec)

  ```

  从服务器上titles表也已经删除。
---