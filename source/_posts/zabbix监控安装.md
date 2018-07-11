---
title: zabbix监控安装
author: Evobot
categories: zabbix
tags:
  - zabbix
  - 监控
abbrlink: 1d9537f2
date: 2018-07-08 22:44:40
image:
---



本文主要介绍了Linux平台常用的监控软件，并重点介绍zabbix监控软件，并且详细演示了如何安装zabbix服务端和客户端以及修改zabbix管理员密码。

<!--more-->

---

# LInux监控平台介绍

- 常见的开源监控软件，有cacti、nagios、zabbix、smokping、open-falcon等等；
- cacti、smokeping偏向于基础监控，成图非常漂亮；
- cacti、nagios、zabbix服务端监控中心，需要php环境支持，其中zabbix和cacti都需要MySQL作为数据存储，nagios不用存储历史数据，注重服务或监控项的状态，zabbix会获取服务或监控项目的数据，会把数据记录到数据库中，从而可以成图；
- open-falcon是小米公司开发，开源后受到很多公司和运维工程师的青睐，适合大企业，本文及后续以介绍zabbix为主。

# zabbix监控

## zabbix介绍

- zabbix采用C/S架构，基于C++开发，监控中心支持web界面配置及管理；

- 单个zabbix节点，可以支持上万台客户端；

- zabbix包含了5个组件；

  - zabbix-server监控中心，接收客户端上报信息，负责配置、统计、操作数据；
  - 数据存储，可以使用MySQL等数据库软件；
  - web UI，在web见面下操作配置是zabbix简单易用的主要原因；
  - zabbix-proxy可选组件，它可以代替zabbix-server的功能，减轻server的压力；
  - zabbix-agent，客户端软件，负责采集各个监控服务或项目的数据，并且上报给zabbix-server

- zabbix的监控流程图如下：

  ![zabbix-flow](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-flow.jpeg)

## 环境准备

- 首先需要准备两台服务器或虚拟机，并到官方下载地址：[www.zabbix.com/download](https://www.zabbix.com/download)，下载对应版本的yum源：

  ```bash
  $ rpm -i https://repo.zabbix.com/zabbix/3.2/rhel/7/x86_64/zabbix-release-3.2-1.el7.noarch.rpm
  ```

  - 执行这一步之后，实际上会在系统内增加一个zabbix的yum源：

  ```bash
  [root@rs2 ~]# cat /etc/yum.repos.d/zabbix.repo 
  [zabbix]
  name=Zabbix Official Repository - $basearch
  baseurl=http://repo.zabbix.com/zabbix/3.2/rhel/7/$basearch/
  enabled=1
  gpgcheck=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

  [zabbix-non-supported]
  name=Zabbix Official Repository non-supported - $basearch 
  baseurl=http://repo.zabbix.com/non-supported/rhel/7/$basearch/
  enabled=1
  gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
  gpgcheck=1

  ```

## 服务端zabbix安装

- 使用yum在服务端安装zabbix：

  ```bash
  $ yum install -y zabbix-agent zabbix-get zabbix-server-mysql zabbix-web zabbix-web-mysql
  ```

  - 客户端只需要安装zabbix-agent。

- 安装zabbix时，会连带安装httpd和php，另外需要自行安装MySQL，可以参考之前的文章：[LAMP 架构介绍及 MySQL 安装](https://www.evobot.cn/post/cd21d578.html)。

- 首先编辑MySQL的配置文件，在[mysqld]中将MySQL的字符集修改为utf8，增加以下配置后启动MySQL：

  ```bash
  character_set_server=utf8
  ```

- 登录MySQL，黄建zabbix数据库，并指定编码为utf8，并且为zabbix的web服务创建一个MySQL用户：

  ```sql
  mysql> create database zabbix character set utf8 collate utf8_bin;

  mysql> grant all on zabbix.* to zabbix@'127.0.0.1' identified by '123456';
  ```

- 退出MySQL命令行后，进入zabbix-server-mysql存放初始化数据库脚本的目录，本文是在`/usr/share/doc/zabbix-server-mysql-3.2.11/`目录下，具体目录名可以在`/usr/share/doc/`目录下查看zabbix*;

- 在这个目录下有一个名为`create.sql.gz`的sql压缩包文件，我们将其解压，并将其导入到之前创建的zabbix数据库中：

  ```bash
  $ gzip -d create.sql.gz

  $ mysql -uroot -p123456 zabbix < create.sql
  ```

  - 或者也可以使用下面的命令直接导入数据库：

  ```bash
  $ zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uroot -p zabbix
  ```

- 然后编辑zabbix的配置文件`/etc/zabbix/zabbix_server.conf`，添加或修改以下配置：

  ```bash
  # 数据库地址，localhost或ip地址，可以为远程MySQL数据库
  DBHost=127.0.0.1
  # zabbix数据库名字
  DBName=zabbix
  # 数据库用户名
  DBUser=zabbix
  # 数据库密码
  DBPassword=123456
  # 数据库端口
  DBPort=3306
  ```

- 完成后，启动`zabbix-server`和`httpd`服务，默认zabbix监听10051端口：

  ```bash
  $ systemctl start zabbix-server
  $ systemctl start httpd
  ```

  - 需要开机启动zabbix-server和httpd，需要执行：

  ```bash
  $ systemctl enable zabbix-server
  $ systemctl enable httpd
  ```

- 如果出现zabbix进程已经启动却未监听端口的情况，可以在`/var/log/zabbix/`目录下查看日志`zabbix_server.log `：

  ```bash
  [root@load-balancer ~]# tail /var/log/zabbix/zabbix_server.log 
    2659:20180708:235208.100 [Z3001] connection to database 'zabbix' failed: [2002] Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
    2659:20180708:235208.100 database is down: reconnecting in 10 seconds
    2659:20180708:235218.100 [Z3001] connection to database 'zabbix' failed: [2002] Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)

  ```

  - 这里看到报错无法连接mysql，说明配置文件内的数据库配置未配置正确。

## 服务端web安装zabbix

- 启动zabbix后，通过浏览器访问zabbix，URL为`ip/zabbix`，ip为zabbix服务器地址，访问后出现以下界面：

  ![zabbix-install](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-install.png)

- 点击Next step，查看`Fail`项：

  ![zabbix-fial](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-fail.png)

  - 这里提示php的date.timezone未配置，需要在php配置文件`/etc/php.ini`中查找timezone配置项：

  ```bash
  # 将其配置为Asia/Shanghai
  date.timezone = Asia/Shanghai
  ```

  - 配置完成后需要重启httpd。

- 所有项均为OK后，继续Next step，配置数据库：

  ![zabbix-db](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-db.png)

- 接着定义zabbix名称：

  ![zabbix-name](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-name.png)

- 最后Next step到Finish，结束安装，进入登录界面，默认的登录用户名/密码为`Admin/zabbix`:

  ![zabbix-login](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-login.png)

- 进入zabbix后，首先需要更改登录用户名和密码，具体步骤为点击菜单**Administration** > **Users** > **Admin**：

  ![zabbix-users](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-users.png)

  - 点击Admin用户，进入用户详情，然后点击**Change password**更改用户密码，同时可以将语言改为中文，Alias则是用户名，也可以自行更改：

  ![zabbix-passwd](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-passwd.png)

- 完成后重新登录，界面就会变为中文：

  ![zabbix-index](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-index.png)

## 客户端安装zabbix

- 客户端上首先下载yum源，然后使用yum安装`zabbix-agent`:

  ```bash
  $ yum install -y zabbix-agent
  ```

- 然后修改客户端zabbix配置文件`/etc/zabbix/zabbix_agentd.conf`，将server端的ip修改为我们server服务器的IP地址:

  ```bash
  # 服务端IP，被动模式
  Server=192.168.67.127
  # 同样填写server端IP，表示开启主动模式
  ServerActive=192.168.67.127
  # 配置客户端名称
  Hostname=evobot-130
  ```

- 配置完成后，启动zabbix客户端服务：

  ```bash
  $ systemctl start zabbix-agent
  ```

- 查看端口，客户端zabbix端口为10050：

  ```bash
  $ netstat -tlnp |grep zabbix
  tcp        0      0 0.0.0.0:10050           0.0.0.0:*               LISTEN      2181/zabbix_agentd  
  tcp6       0      0 :::10050                :::*                    LISTEN      2181/zabbix_agentd  

  ```

## 忘记Admin密码怎么做

- 前面我们修改了zabbix的Admin用户的密码，如果忘记了密码，可以对密码进行重置；

- 首先在server端登录mysql，并进入zabbix库，zabbix用户密码就存在users表内：

  ```sql
  mysql> desc users;
  +----------------+---------------------+------+-----+---------+-------+
  | Field          | Type                | Null | Key | Default | Extra |
  +----------------+---------------------+------+-----+---------+-------+
  | userid         | bigint(20) unsigned | NO   | PRI | NULL    |       |
  | alias          | varchar(100)        | NO   | UNI |         |       |
  | name           | varchar(100)        | NO   |     |         |       |
  | surname        | varchar(100)        | NO   |     |         |       |
  | passwd         | char(32)            | NO   |     |         |       |
  | url            | varchar(255)        | NO   |     |         |       |
  | autologin      | int(11)             | NO   |     | 0       |       |
  | autologout     | int(11)             | NO   |     | 900     |       |
  | lang           | varchar(5)          | NO   |     | en_GB   |       |
  | refresh        | int(11)             | NO   |     | 30      |       |
  | type           | int(11)             | NO   |     | 1       |       |
  | theme          | varchar(128)        | NO   |     | default |       |
  | attempt_failed | int(11)             | NO   |     | 0       |       |
  | attempt_ip     | varchar(39)         | NO   |     |         |       |
  | attempt_clock  | int(11)             | NO   |     | 0       |       |
  | rows_per_page  | int(11)             | NO   |     | 50      |       |
  +----------------+---------------------+------+-----+---------+-------+
  16 rows in set (0.00 sec)

  ```

- 修改密码，执行下面的sql语句，zabbix的密码采用md5加密，另外如果更改过Admin的名字的话，对应的Admin也需要改为修改后的用户名：

  ```sql
  mysql> update users set passwd=md5('evobot') where alias='Admin';
  ```

- 然后使用新密码登录zabbix即可。

---

