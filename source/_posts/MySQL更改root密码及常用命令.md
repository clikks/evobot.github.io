---
title: MySQL更改root密码及常用命令
author: Evobot
date: 2018-06-19 21:32:22
categories: MySQL
tags: 
  - Linux
  - MySQL
  - Centos
image:
---



本文主要介绍MySQL的常用操作，比如怎样更改MySQL的root密码，怎样连接MySQL，以及MySQL的常用的一些命令。

<!--more-->

---

# 更改root密码

- 登录MySQL的命令是`/usr/local/mysql/bin/mysql`命令，由于这个目录不在系统PATH内，所以我们可以将目录加入PATH或者使用软连接的方式。
  ```bash
  export PATH=$PATH:/usr/local/mysql/bin/
  #也可以将上面的命令添加到/etc/profile中

  ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql
  ```

- 在MySQL5.7以下版本，默认安装后是没有密码的，可以直接使用`mysql -uroot`登录；

- 设置MySQL的密码，可以使用下面的命令：

  ```bash
  [root@evobot mysql]# mysqladmin -uroot password '123456'
  Warning: Using a password on the command line interface can be insecure.
  ```

  > 然后登录MySQL需要使用命令`mysql -uroot -p123456`。

- 如果对已经存在的密码进行更改，则需要使用下面的命令形式：

  ```bash
  [root@evobot mysql]# mysqladmin -uroot -p123456 password '778899'
  Warning: Using a password on the command line interface can be insecure.
  ```

  > 如果密码中存在特殊符号，则`-p`后面的密码需要使用单引号括起来。

- 如果忘记了MySQL的root密码，那么更改密码就需要先编辑配置文件，在`[mysqld]`下增加一行`skip-grant`，表示忽略鉴权;

  - 更改了配置文件后，需要重启MySQL生效，然后即可无密码登录到MySQL中；
  - 然后在MySQL的命令行中更改密码，更改密码需要修改mysql数据库：

  ```sql
  mysql> use mysql;
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A

  Database changed

  mysql> select Password from user where User='root';	#查看user表中存储的root密码
  +-------------------------------------------+
  | Password                                  |
  +-------------------------------------------+
  | *DC6311B4273794D47DDAECDA5DB63206132137C2 |
  |                                           |
  |                                           |
  |                                           |
  +-------------------------------------------+
  4 rows in set (0.00 sec)

  mysql> update user set password=password('123456') where user='root';
  Query OK, 4 rows affected (0.00 sec)
  Rows matched: 4  Changed: 4  Warnings: 0
  ```

- 上面使用了`update user set password=password('123456') where user='root'`命令将root密码更改，更改完成密码之后，还需要将之前修改的my.cnf添加的`skip-grant`配置删除，然后重启MySQL服务，即可使用新密码登录。

- 在MySQL5.7中，因为安装时会设置一个随机密码，这个密码保存在家目录的`.mysql_secret`文件内，登录MySQL5.7之后更改密码，使用下面的命令：

  ```sql
  mysql> use mysql

  mysql> SET PASSWORD FOR 'root'@localhost = PASSWORD ('123456');

  ```

  ​

# 登录MySQL

- 常用的登录MySQL的命令有下面几种：
  - `mysql -uroot -p123456`：在本机使用root账户密码登录mysql；
  - `mysql -uroot -p123456 -h127.0.0.1 -P3306`：使用ip地址连接远程的MySQL服务，`-h`指定host，`-P`指定端口；
  - `mysql -uroot -p123456 -S/tmp/mysql.sock`：使用`-S`指定MySQL的socket登录本机的MySQL服务；
  - `mysql -uroot -p123456 -e "show databases"`：登录MySQL服务并使用`-e`选项执行SQL语句，常用在shell脚本内，执行语句后会退出MySQL回到bash环境。

# MySQL常用命令

- 查询数据库列表：`show databases;`

  ```sql
  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | mysql              |
  | performance_schema |
  | test               |
  +--------------------+
  4 rows in set (0.00 sec)

  ```

- 切换到指定数据库：`use db_name;`

  ```sql
  mysql> use mysql;
  Reading table information for completion of table and column names
  You can turn off this feature to get a quicker startup with -A

  Database changed
  ```

- 查看当前数据库中的表：`show tables;`

  ```sql
  mysql> show tables;
  +---------------------------+
  | Tables_in_mysql           |
  +---------------------------+
  | columns_priv              |
  | db                        |
  | event                     |
  | func                      |
  | general_log               |
  | help_category             |
  | help_keyword              |
  | user                      |
  +---------------------------+
  28 rows in set (0.00 sec)

  ```

- 查询表中的字段：`desc tb_name;`

  ```sql
  mysql> desc user;
  +------------------------+-----------------------------------+------+-----+-----------------------+-------+
  | Field                  | Type                              | Null | Key | Default               | Extra |
  +------------------------+-----------------------------------+------+-----+-----------------------+-------+
  | Host                   | char(60)                          | NO   | PRI |                       |       |
  | User                   | char(16)                          | NO   | PRI |                       |       |
  | Password               | char(41)                          | NO   |     |                       |       |
  | password_expired       | enum('N','Y')                     | NO   |     | N                     |       |
  +------------------------+-----------------------------------+------+-----+-----------------------+-------+
  43 rows in set (0.00 sec)

  ```

  > 最左边的列就是字段名，右侧则是字段的格式定义，如整数，字符。

- 查看创建表的SQL语句：`show create table user\G;`，`\G`表示竖排显示：

  ```sql
  mysql> show create table user\G;
  *************************** 1. row ***************************
         Table: user
  Create Table: CREATE TABLE `user` (
    `Host` char(60) COLLATE utf8_bin NOT NULL DEFAULT '',
    `User` char(16) COLLATE utf8_bin NOT NULL DEFAULT '',
    `Password` char(41) CHARACTER SET latin1 COLLATE latin1_bin NOT NULL DEFAULT '',
    `Select_priv` enum('N','Y') CHARACTER SET utf8 NOT NULL DEFAULT 'N',
    PRIMARY KEY (`Host`,`User`)
  ) ENGINE=MyISAM DEFAULT CHARSET=utf8 COLLATE=utf8_bin COMMENT='Users and global privileges'
  1 row in set (0.00 sec)

  ERROR: 
  No query specified

  ```



- 查看当前登录用户：`select user();`

  ```sql
  # 远程登录时显示主机名
  mysql> select user();
  +-------------+
  | user()      |
  +-------------+
  | root@evobot |
  +-------------+
  1 row in set (0.00 sec)

  # 本地登录
  mysql> select user();
  +----------------+
  | user()         |
  +----------------+
  | root@localhost |
  +----------------+
  1 row in set (0.00 sec)

  ```

  ​

- MySQL的命令行同样可以记录命令历史，而记录命令历史的文件为家目录下的`.mysql_history`；



- 查看当前使用的数据库：`select database();`

  ```sql
  mysql> select database();
  +------------+
  | database() |
  +------------+
  | mysql      |
  +------------+
  1 row in set (0.00 sec)
  ```

- 创建数据库：`create database db_name;`

  ```sql
  mysql> create database evobot;
  Query OK, 1 row affected (0.11 sec)

  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | evobot             |
  | mysql              |
  | performance_schema |
  | test               |
  +--------------------+
  5 rows in set (0.00 sec)

  ```

- 在数据库中创建表：```create table tb_name(`id` int(4), `name` char(40));```

  ```sql
  mysql> use evobot
  Database changed

  mysql> create table test1(`id` int(4), `name` char(40));
  Query OK, 0 rows affected (0.13 sec)

  mysql> show tables;
  +------------------+
  | Tables_in_evobot |
  +------------------+
  | test1            |
  +------------------+
  1 row in set (0.00 sec)

  mysql> show create table test1\G;
  *************************** 1. row ***************************
         Table: test1
  Create Table: CREATE TABLE `test1` (
    `id` int(4) DEFAULT NULL,
    `name` char(40) DEFAULT NULL
  ) ENGINE=InnoDB DEFAULT CHARSET=latin1
  1 row in set (0.00 sec)

  ERROR: 
  No query specified

  ```

  > 这里可以看到表的ENGINE为InnoDB，字符为latin1，如果想指定表的ENGINE和字符集，在创建表时，使用```create table t1(`id` int(4), `name` char(40)) ENGINE=InnoDB DEFAULT CHARSET=utf8;```

  - 关于MySQL的myisam和innodb引擎的优劣，可以查看这篇博文：[MySQL存储引擎MyISAM与InnoDB的优劣](https://www.pureweber.com/article/myisam-vs-innodb/)

- 查看当前数据库版本：`select versions()`

  ```sql
  mysql> select version();
  +-----------+
  | version() |
  +-----------+
  | 5.6.36    |
  +-----------+
  1 row in set (0.00 sec)

  ```



- 查看数据库状态：`show status;`

- 查看my.cnf中定义的各种参数：`show variables;`，`show variables like 'max_connect%';`

  ```sql
  mysql> show variables like 'max_connect%';
  +--------------------+-------+
  | Variable_name      | Value |
  +--------------------+-------+
  | max_connect_errors | 100   |
  | max_connections    | 151   |
  +--------------------+-------+
  2 rows in set (0.00 sec)

  mysql> show variables like 'slow%';
  +---------------------+---------------------------------------------+
  | Variable_name       | Value                                       |
  +---------------------+---------------------------------------------+
  | slow_launch_time    | 2                                           |
  | slow_query_log      | OFF                                         |
  | slow_query_log_file | /usr/local/mysql/data/mysql/evobot-slow.log |
  +---------------------+---------------------------------------------+
  3 rows in set (0.00 sec)

  ```



- 修改MySQL中的参数：`set global max_connect_errors=1000`

  ```sql
  mysql> set global max_connect_errors=1000;
  Query OK, 0 rows affected (0.00 sec)

  mysql> show variables like 'max_connect%';
  +--------------------+-------+
  | Variable_name      | Value |
  +--------------------+-------+
  | max_connect_errors | 1000  |
  | max_connections    | 151   |
  +--------------------+-------+
  2 rows in set (0.00 sec)

  ```

  > 这种方式只是临时生效，如果需要永久生效，则需要更改my.cnf对应的参数值。



- 查看队列：`show processlist`，`show full processlist`，这两天命令的区别在于，full会完整的列出队列的信息：

  ```sql
  mysql> show processlist;
  +----+------+--------------+--------+---------+------+-------+------------------+
  | Id | User | Host         | db     | Command | Time | State | Info             |
  +----+------+--------------+--------+---------+------+-------+------------------+
  |  5 | root | evobot:56428 | evobot | Query   |    0 | init  | show processlist |
  +----+------+--------------+--------+---------+------+-------+------------------+
  1 row in set (0.00 sec)

  mysql> show full processlist;
  +----+------+--------------+--------+---------+------+-------+-----------------------+
  | Id | User | Host         | db     | Command | Time | State | Info                  |
  +----+------+--------------+--------+---------+------+-------+-----------------------+
  |  5 | root | evobot:56428 | evobot | Query   |    0 | init  | show full processlist |
  +----+------+--------------+--------+---------+------+-------+-----------------------+
  1 row in set (0.00 sec)

  ```

  ​