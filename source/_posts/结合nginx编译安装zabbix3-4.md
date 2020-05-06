---
title: 结合nginx编译安装zabbix3.4
author: Evobot
categories: zabbix
tags:
  - zabbix
  - 监控
abbrlink: 7a0f12f0
date: 2018-07-11 11:32:00
image:
---

本文主要演示如何编译安装zabbix3.4，并采用LNMP为zabbix提供web访问的相关配置。
<!--more-->

# 编译安装

- 首先需要安装php环境，编译安装php时，需要增加bcmath、ldap、sockets、gettext模块，安装ldap时需要安装openldap-devel依赖包，并执行`cp -frp /usr/lib64/libldap* /usr/lib/`命令；或者编译时增加`--enable-bcmath --enable-sockets --with-gettext --with-ldap=shared`;

- 下载zabbix源码包并解压，并安装依赖包`net-snmp-devel`、`libevent-devel`；

- 进入zabbix源码目录，执行编译安装，编译参数如下：

  ```bash
  ./configure --prefix=/usr/local/zabbix-3.4.11/ --enable-server --enable-agent --with-mysql --with-net-snmp --with-libcurl --with-libxml2
  
  make && make install
  ```

  > 如果需要开启zabbix-proxy功能，可以增加`--enable-proxy`参数。

- 添加zabbix用户：

  ```bash
  useradd -s /sbin/nologin -M zabbix
  ```

- 为zabbix创建数据库、设置字符集，并为其创建MySQL登陆用户，在MySQL命令行内执行下面的SQL语句：

  ```sql
  create database zabbix default character set utf8 collate utf8_bin;
   
  grant all on zabbix.* to 'zabbix'@'localhost' identified by '123456';
  ```

- 然后导入zabbix提供的数据库sql文件，sql文件在zabbix源码包的`database`目录内，根据使用的数据库选择对应的数据库目录，这里使用MySQL，zabbix提供了三个sql文件，分别是`data.sql`、`images.sql`、`schema.sql`，如果是proxy服务，则只需要导入schema.sql，zabbix-server则需要导入全部的三个数据库文件：

  ```bash
  mysql -uroot -p123456 zabbix < database/mysql/schema.sql
  mysql -uroot -p123456 zabbix < database/mysql/images.sql
  mysql -uroot -p123456 zabbix < database/mysql/data.sql
  ```

  > 需要注意导入的数据库文件顺序不能错，否则会报错。

# 配置zabbix

## 服务端配置

- 编译安装的zabbix，配置文件在安装目录下的`etc`目录中：

  ```bash
  $ ls /usr/local/zabbix-3.4.11/etc/
  zabbix_agentd.conf  zabbix_agentd.conf.d  zabbix_server.conf  zabbix_server.conf.d
  ```

- 修改`zabbix_server.conf`配置文件中下面几个参数：

  ```bash
  DBHost=localhost
  DBName=zabbix
  DBUser=zabbix
  DBPassword=123456
  DBPort=3306
  ```

- 然后从源码包复制启动脚本到`/etc/init.d`目录下：

  ```
  cp /usr/local/src/zabbix-3.4.11/misc/init.d/fedora/core5/* /etc/init.d/
  ```

- 修改启动脚本里zabbix启动命令路径：

  ```bash
  # 修改/etc/init.d/zabbix_server
  ZABBIX_BIN="/usr/local/sbin/zabbix_server"
  修改为
  ZABBIX_BIN="/usr/local/zabbix-3.4.11/sbin/zabbix_server"
  
  # 修改/etc/init.d/zabbix_agentd
  ZABBIX_BIN="/usr/local/sbin/zabbix_agentd"
  修改为
  ZABBIX_BIN="/usr/local/zabbix-3.4.11/sbin/zabbix_agentd"
  ```

- 启动zabbix，出现报错`libmysqlclient.so.20`不存在：

  ```bash
  [root@evobot zabbix-3.4.11]# systemctl status zabbix_server -l
  ● zabbix_server.service - SYSV: Zabbix Monitoring Server
     Loaded: loaded (/etc/rc.d/init.d/zabbix_server; bad; vendor preset: disabled)
     Active: failed (Result: exit-code) since 二 2018-07-10 15:51:15 CST; 4s ago
       Docs: man:systemd-sysv-generator(8)
    Process: 21543 ExecStart=/etc/rc.d/init.d/zabbix_server start (code=exited, status=1/FAILURE)
  
  7月 10 15:51:15 evobot systemd[1]: Starting SYSV: Zabbix Monitoring Server...
  7月 10 15:51:15 evobot zabbix_server[21543]: Starting Zabbix Server: /usr/local/zabbix-3.4.11/sbin/zabbix_server: error while loading shared libraries: libmysqlclient.so.20: cannot open shared object file: No such file or directory
  7月 10 15:51:15 evobot zabbix_server[21543]: [失败]
  7月 10 15:51:15 evobot systemd[1]: zabbix_server.service: control process exited, code=exited status=1
  7月 10 15:51:15 evobot systemd[1]: Failed to start SYSV: Zabbix Monitoring Server.
  7月 10 15:51:15 evobot systemd[1]: Unit zabbix_server.service entered failed state.
  7月 10 15:51:15 evobot systemd[1]: zabbix_server.service failed.
  
  ```

  - 这是因为我的MySQL不是yum安装，所以需要执行下面的命令：

  ```bash
  ln -s /usr/local/mysql/lib/libmysqlclient.so.20 /usr/lib64/libmysqlclient.so.20
  ```

- 默认编译安装的zabbix的日志文件在`/tmp`目录下，使用`service zabbix_server start`启动zabbix。


## 虚拟主机配置

- 这里我使用nginx作为zabbix的web服务，安装nginx的步骤不再演示，在nginx虚拟主机目录创建新的虚拟主机配置文件`zabbix.conf`，内容如下：

  ```bash
  server
  {
      listen 80;
      server_name monitor.evobot.cn;
      index index.html index.htm index.php;
      root /var/www/zabbix;
      access_log /tmp/zabbix.log combined_realip;
  
      location ~ .php
      {
          include fastcgi_params;
          fastcgi_pass unix:/tmp/php-fcgi.sock;
          fastcgi_index index.php;
          set $path_info "";
          set $real_script_name $fastcgi_script_name;
          if ($fastcgi_script_name ~ "^(.+?\.php)(/.+)$")
          {
              set $real_script_name $1;
              set $path_info $2;
          }
          fastcgi_param SCRIPT_FILENAME /var/www/wordpress$fastcgi_script_name;
          fastcgi_param SCRIPT_NAME $real_script_name;
          fastcgi_param PATH_INFO $path_info;
      }
  
      location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
      {
          expires 7d;
          access_log off;
      }
  
      location ~ .*\.(js|css)$
      {
          expires 12h;
          access_log off;
      }
  }
  
  ```

- 拷贝zabbix的前端文件到nginx中配置的虚拟主机目录`/var/www/zabbix`下，zabbix的前端文件位于源码包的`frontends/php/`目录下：

  ```bash
  $ mkdir /var/www/zabbix
  $ cp -rp /usr/local/src/zabbix-3.4.11/frontends/php/* /var/www/zabbix/
  
  ```

- 完成后使用浏览器访问zabbix安装界面即可，在安装的最后一步时，会提示需要下载`zabbix.conf.php`文件，按提示下载并上传到`/var/www/zabbix/conf`完成安装。