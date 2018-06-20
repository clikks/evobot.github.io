---
title: MySQL用户管理、常用SQL语句及数据库备份与恢复
author: Evobot
date: 2018-06-20 20:52:32
categories: MySQL
tags:
  - Linux
  - Centos
  - MySQL
image:
---



本文主要介绍MySQL的用户管理、常用的SQL语句，以及如何进行数据库的备份和恢复作业。

<!--more-->

---

# MySQL用户管理

## 创建用户及授权

### 针对所有权限

- 为了保证安全性，我们不能将MySQL的root用户密码给每个开发人员，所以需要单独创建用户并加以授权，让指定的用户只能操作指定的数据库来保证安全；

- 创建用户使用下面的命令：

  ```sql
  grant all on *.* to 'username'@'127.0.0.1' identified by 'password';
  ```

  - 这里`grant`表示授权，`all`是指所有的操作，如删除、创建、修改等;
  - `*.*`表示所有的库和表，第一个`*`指库名，后面一个`*`表示表名，`*.*`则表示所有库的所有表;
  - `‘user1'@'127.0.0.1'`是指创建user1用户，并且user1只能通过127.0.0.1这个来源ip登录MySQL，也可以写为`%`表示所有的ip;
  - `identified by`则是指定user1的密码。

- 由于在创建用户时指定了来源ip，所以登录时必须使用`-h`选项指定host，因为默认登录使用的是MySQL的socket进行连接：

  ```bash
  [root@evobot ~]# mysql -uuser1 -p
  Enter password: 
  ERROR 1045 (28000): Access denied for user 'user1'@'localhost' (using password: YES)

  [root@evobot ~]# mysql -uuser1 -p -h127.0.0.1
  Enter password: 
  Welcome to the MySQL monitor. 
  ...
  ```

- 如果想要使用户登录MySQL使用socket连接，那么在指定host时，写为`localhost`即可，如下：

  ```sql
  grant all on *.* to 'username'@'localhost' identified by 'password';
  ```

  示例：

  ```bash
  [root@evobot ~]# mysql -uuser5 -p123456
  Warning: Using a password on the command line interface can be insecure.
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  mysql> 
  ```

### 针对指定权限、数据库

- 针对指定的数据库授权指定的权限，使用下面的SQL语句：

  ```sql
  grant SELECT,UPDATE,INSERT on db_name.* to 'username'@'localhost' identified by 'password';
  ```

  示例：

  ```sql
  mysql> grant SELECT,UPDATE,INSERT on test.* to 'user6'@'localhost' identified by 'password';
  Query OK, 0 rows affected (0.00 sec)

  [root@evobot ~]# mysql -uuser6 -p
  Enter password: 

  mysql> show databases;
  +--------------------+
  | Database           |
  +--------------------+
  | information_schema |
  | test               |
  +--------------------+
  2 rows in set (0.00 sec)

  mysql> drop database test;
  ERROR 1044 (42000): Access denied for user 'user6'@'localhost' to database 'test'
  ```

  - 限定用户只能操作test数据库，所以用户登录后只能看到test数据库，并且由于用户没有drop权限，所以操作会被拒绝。

### 查看权限

- 对于已经登录的当前用户，使用`show grant;`可以查看当前用户的权限：

  ```sql
  mysql> select user();
  +-----------------+
  | user()          |
  +-----------------+
  | user6@localhost |
  +-----------------+
  1 row in set (0.00 sec)

  mysql> show grants;
  +---------------------------------------------------------------------------+
  | Grants for user6@localhost                                                |
  +---------------------------------------------------------------------------+
  | GRANT USAGE ON *.* TO 'user6'@'localhost' IDENTIFIED BY PASSWORD <secret> |
  | GRANT SELECT, INSERT, UPDATE ON `test`.* TO 'user6'@'localhost'           |
  +---------------------------------------------------------------------------+
  2 rows in set (0.00 sec)
  ```

- 对于具有操作名为mysql数据库的用户，查看其他用户的权限，使用的sql语句如下：

  ```sql
  show grants for user3@'%';
  ```

  示例：

  ```sql
  mysql> select user();
  +-----------------+
  | user()          |
  +-----------------+
  | user5@localhost |
  +-----------------+
  1 row in set (0.00 sec)

  mysql> show grants for user3@'%';
  +---------------------------------------------------------------------------------------------------------------+
  | Grants for user3@%                                                                                            |
  +---------------------------------------------------------------------------------------------------------------+
  | GRANT ALL PRIVILEGES ON *.* TO 'user3'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
  +---------------------------------------------------------------------------------------------------------------+
  1 row in set (0.00 sec)

  ```

- 上面两种`show grants`命令所列出的权限信息，可以方便我们对用户的权限进行更改，如在不知道用户密码的情况下，需要更改用户的源IP，那么在root用户下可以直接复制列出的语句，把其中的IP更改执行即可：

  ```sql
  mysql> show grants for user5@'localhost';
  +-----------------------------------------------------------------------------------------------------------------------+
  | Grants for user5@localhost                                                                                            |
  +-----------------------------------------------------------------------------------------------------------------------+
  | GRANT ALL PRIVILEGES ON *.* TO 'user5'@'localhost' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
  +-----------------------------------------------------------------------------------------------------------------------+
  1 row in set (0.00 sec)

  mysql>  GRANT ALL PRIVILEGES ON *.* TO 'user5'@'127.0.0.1' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9';
  Query OK, 0 rows affected (0.00 sec)

  mysql> show grants for user5@'127.0.0.1';
  +-----------------------------------------------------------------------------------------------------------------------+
  | Grants for user5@127.0.0.1                                                                                            |
  +-----------------------------------------------------------------------------------------------------------------------+
  | GRANT ALL PRIVILEGES ON *.* TO 'user5'@'127.0.0.1' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
  +-----------------------------------------------------------------------------------------------------------------------+
  1 row in set (0.00 sec)
  ```

# 常用SQL语句

- 操作MySQL的SQL主要就是增删改查，包括`select`、`insert`、`update`、`drop`等；

## 查询语句

- `select count(*) from mysql.user;`，其中count(*)是统计行数，即统计mysql库中user表一共有多少条数据：

  ```sql
  mysql> select count(*) from mysql.user;
  +----------+
  | count(*) |
  +----------+
  |       15 |
  +----------+
  1 row in set (0.00 sec)

  ```

- `select * from mysql.db;`，查看mysql库的db表中所有的数据，一般不建议使用，查询所有数据非常耗费资源；

- `select db from mysql.db;`，查看mysql.db表中的db字段，字段名可以有多个：

  ```sql
  mysql> select db,user from mysql.db;
  +---------+-------+
  | db      | user  |
  +---------+-------+
  | test    |       |
  | test\_% |       |
  | test    | user6 |
  +---------+-------+
  3 rows in set (0.00 sec)

  ```

- `select * from mysql.db where host like 'local%'\G;`:

  ```sql
  mysql> select * from mysql.db where host like 'local%';
  +-----------+------+-------+-------------+-------------+-------------+-------------+-------------+-----------+------------+-----------------+------------+------------+-----------------------+------------------+------------------+----------------+---------------------+--------------------+--------------+------------+--------------+
  | Host      | Db   | User  | Select_priv | Insert_priv | Update_priv | Delete_priv | Create_priv | Drop_priv | Grant_priv | References_priv | Index_priv | Alter_priv | Create_tmp_table_priv | Lock_tables_priv | Create_view_priv | Show_view_priv | Create_routine_priv | Alter_routine_priv | Execute_priv | Event_priv | Trigger_priv |
  +-----------+------+-------+-------------+-------------+-------------+-------------+-------------+-----------+------------+-----------------+------------+------------+-----------------------+------------------+------------------+----------------+---------------------+--------------------+--------------+------------+--------------+
  | localhost | test | user6 | Y           | Y           | Y           | N           | N           | N         | N          | N               | N          | N          | N                     | N                | N                | N              | N                   | N                  | N            | N          | N            |
  +-----------+------+-------+-------------+-------------+-------------+-------------+-------------+-----------+------------+-----------------+------------+------------+-----------------------+------------------+------------------+----------------+---------------------+--------------------+--------------+------------+--------------+
  1 row in set (0.00 sec)

  ```

## 增加语句

- `insert into db_name.tb_name values (value1,value2);`，向指定的数据表内插入一条数据：

  ```sql
  mysql> insert into evobot.test1 values (1,'lilei');
  Query OK, 1 row affected (0.02 sec)

  mysql> select * from test1;
  +------+-------+
  | id   | name  |
  +------+-------+
  |    1 | lilei |
  +------+-------+
  1 row in set (0.00 sec)

  ```

  > 插入数据的时候，数据的格式要符合表的字段定义，如字符，就需要使用单引号。

## 修改语句

- `update db_name.tb_name set field='new_value' where filed=1;`，表示修改指定表中指定字段的值：

  ```sql
  mysql> update evobot.test1 set name='hanmeimei' where id=1;
  Query OK, 1 row affected (0.01 sec)
  Rows matched: 1  Changed: 1  Warnings: 0

  mysql> select * from test1;
  +------+-----------+
  | id   | name      |
  +------+-----------+
  |    1 | hanmeimei |
  +------+-----------+
  1 row in set (0.00 sec)

  ```

  > 如果where筛选出的有多条数据，那么筛选出来的数据都会被修改。

## 删除语句

- `delete from db_name.tb_name where filed=value;`，删除指定表中的指定的数据：

  ```sql
  mysql> delete from evobot.test1 where id=1;
  Query OK, 1 row affected (0.01 sec)

  mysql> select * from test1;
  Empty set (0.00 sec)

  ```

- `truncate db_name.tb_name;`，清空指定的表的所有内容：

  ```sql
  mysql> select * from test1;
  +------+-----------+
  | id   | name      |
  +------+-----------+
  |    1 | lilei     |
  |    2 | hanmeimei |
  +------+-----------+
  2 rows in set (0.00 sec)

  mysql> truncate evobot.test1;
  Query OK, 0 rows affected (0.08 sec)

  mysql> select * from test1;
  Empty set (0.00 sec)

  mysql> desc test1;
  +-------+----------+------+-----+---------+-------+
  | Field | Type     | Null | Key | Default | Extra |
  +-------+----------+------+-----+---------+-------+
  | id    | int(4)   | YES  |     | NULL    |       |
  | name  | char(40) | YES  |     | NULL    |       |
  +-------+----------+------+-----+---------+-------+
  2 rows in set (0.00 sec)

  ```

- `drop table db_name.tb_name;`，表示删除指定的数据表，`drop database db_name;`表示删除数据库：

  ```sql
  mysql> drop table test1;
  Query OK, 0 rows affected (0.03 sec)

  mysql> show tables;
  Empty set (0.00 sec)

  mysql> drop database evobot;
  Query OK, 0 rows affected (0.00 sec)

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

- **删除、清空数据库或表的操作一定要谨慎，一旦删错而又没有备份，会导致严重后果！**

# 数据库备份恢复

- 备份数据库，是非常重要的操作，能够保证我们的数据库在出现问题时，尽快恢复到之前的状态和数据；

## 备份操作

- `mysqldump -uroot -p db_name > /path/to/db_backup.sql`，表示备份指定的数据库：

  ```bash
  [root@evobot ~]# mysqldump -uroot -p123456 mysql > /tmp/mysql_bak.sql
  Warning: Using a password on the command line interface can be insecure.
  ```

- 备份表的操作，相比备份数据库，只需要在库名后加上表名即可，命令为`mysqldump -uroot -p db_name tb_name > /path/to/backup.sql`：

  ```bash
  mysqldump -uroot -p evobot test1 > /tmp/tb_test1.sql
  ```

- 备份所以的数据库，使用命令`mysqldump -uroot -p -A /path/to/bak.sql`：

  ```bash
  mysqldump -uroot -p -A > /tmp/mysql_A.sql
  ```

- 只备份表的结构，是指不备份数据库表的数据内容，只备份表的结构，即备份文件中没有insert语句，命令为`mysqldump -uroot -p -d db_name > /path/to/bak.sql`：

  ```bash
  [root@evobot ~]# mysqldump -uroot -p -d evobot > /tmp/evobot.sql
  Enter password: 
  [root@evobot ~]# cat /tmp/evobot.sql 
  -- MySQL dump 10.13  Distrib 5.6.36, for linux-glibc2.5 (x86_64)
  --
  -- Host: localhost    Database: evobot
  -- ------------------------------------------------------
  -- Server version	5.6.36

  /*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
  /*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
  /*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
  /*!40101 SET NAMES utf8 */;
  /*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
  /*!40103 SET TIME_ZONE='+00:00' */;
  /*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
  /*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
  /*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
  /*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

  --
  -- Table structure for table `test1`
  --

  DROP TABLE IF EXISTS `test1`;
  /*!40101 SET @saved_cs_client     = @@character_set_client */;
  /*!40101 SET character_set_client = utf8 */;
  CREATE TABLE `test1` (
    `id` int(4) DEFAULT NULL,
    `name` char(40) DEFAULT NULL
  ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
  /*!40101 SET character_set_client = @saved_cs_client */;
  /*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

  /*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
  /*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
  /*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
  /*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
  /*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
  /*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
  /*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

  -- Dump completed on 2018-06-20 23:41:10
  ```

  ​

## 恢复操作

- `mysqldump -uroot -p db_name < /path/to/backup.sql`，表示将备份sql文件恢复到指定库中：

  ```bash
  [root@evobot ~]# mysql -uroot -p123456 -e "create database mysql2"

  [root@evobot ~]# mysql -uroot -p mysql2 < /tmp/mysql_bak.sql 

  # 登录并直接进入mysql2数据库
  [root@evobot ~]# mysql -uroot -p mysql2
  ```

  ```sql
  mysql> select database();
  +------------+
  | database() |
  +------------+
  | mysql2     |
  +------------+
  1 row in set (0.00 sec)

  mysql> show tables;
  +---------------------------+
  | Tables_in_mysql2          |
  +---------------------------+
  | columns_priv              |
  | db                        |
  | event                     |
  | func                      |
  | general_log               |
  | help_category             |
  | help_keyword              |
  | help_relation             |
  | help_topic                |
  | innodb_index_stats        |
  | innodb_table_stats        |
  | ndb_binlog_index          |
  | plugin                    |
  | proc                      |
  | procs_priv                |
  | proxies_priv              |
  | servers                   |
  | slave_master_info         |
  | slave_relay_log_info      |
  | slave_worker_info         |
  | slow_log                  |
  | tables_priv               |
  | time_zone                 |
  | time_zone_leap_second     |
  | time_zone_name            |
  | time_zone_transition      |
  | time_zone_transition_type |
  | user                      |
  +---------------------------+
  28 rows in set (0.00 sec)

  ```

- 恢复备份的数据表，与恢复数据库相同，指定数据库名即可；

