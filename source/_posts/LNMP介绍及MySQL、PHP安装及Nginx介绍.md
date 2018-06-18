---
title: LNMP介绍及MySQL、PHP安装及Nginx介绍
author: Evobot
date: 2018-06-06 21:51:33
categories: LNMP
tags: [Linux, Centos]
image:
---



本文主要介绍LNMP架构和LNMP架构中MySQL、PHP的安装，以及Nginx软件包介绍。

<!--more-->

---

# LNMP架构

- LNMP与LAMP类似，只是apache换成了nginx，LNMP的php是作为独立服务存在，叫做php-fpm，Nginx会将动态请求转发给php-fpm进行处理，Nginx直接处理静态的请求，Nginx对静态文件请求的并发数可以上万；

- LNMP的架构如下图所示：

  ![LNMP](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/LNMP.png)

# MySQL、PHP安装

- MySQL的安装与LAMP中MySQL的安装相同，这里不再演示,可以查看博文[LAMP 架构介绍及 MySQL 安装](https://www.evobot.cn/post/cd21d578.html)；

- PHP的安装与LAMP不同，需要开启php-fpm服务；

- 首先进入php源码包目录，源码包同样使用LAMP的源码包版本相同，执行一次`make clean`清除之前编译的缓存；

- 增加php-fpm用户，然后进行编译：

  ```bash
  useradd -s /sbin/nologin -M php-fpm
  
  ./configure --prefix=/usr/local/php-fpm \
  --with-config-file-path=/usr/local/php-fpm/etc \
  --enable-fpm \
  --with-fpm-user=php-fpm \
  --with-fpm-group=php-fpm \
  --with-mysql=/usr/local/mysql \
  --with-mysqli=/usr/local/mysql/bin/mysql_config \
  --with-pdo-mysql=/usr/local/mysql \
  --with-mysql-sock=/tmp/mysql.sock \
  --with-libxml-dir \
  --with-gd \
  --with-jpeg-dir \
  --with-png-dir \
  --with-freetype-dir \
  --with-iconv-dir \
  --with-zlib-dir \
  --with-mcrypt \
  --enable-soap \
  --enable-gd-native-ttf \
  --enable-ftp \
  --enable-mbstring \
  --enable-exif \
  --with-pear \
  --with-curl \
  --with-openssl
  
  make && make install
  ```

  > 可能会产生报错需要安装libcurl-devel，openssl-devel。

- 安装完成后进入php-fpm目录，其中在sbin目录下有一个php-fpm执行文件，这个文件和bin/php同样都可以执行`-i`、`-m`选项查看php信息和模块信息，但是php-fpm可以使用`-t`选项来测试php配置文件是否正确：

  ```bash
  [root@evobot php-fpm]# sbin/php-fpm -t
  [06-Jun-2018 23:16:55] ERROR: failed to open configuration file '/usr/local/php-fpm/etc/php-fpm.conf': No such file or directory (2)
  [06-Jun-2018 23:16:55] ERROR: failed to load configuration file '/usr/local/php-fpm/etc/php-fpm.conf'
  [06-Jun-2018 23:16:55] ERROR: FPM initialization failed
  ```

- 然后从源码包复制配置文件到`/usr/local/php-fpm/etc`目录下，在源码包内，有两个配置文件，分别是**php.ini-development**开发配置，和**php.ini-production**生产配置，主要区别在日志级别的不同：

  ```bash
  [root@evobot php-5.6.32]# cp php.ini-production /usr/local/php-fpm/etc/php.ini
  ```

- 然后在/usr/local/php-fpm/etc目录下创建`php-fpm.conf`，并写入以下内容：

  ```bash
  [global]
  pid = /usr/local/php-fpm/var/run/php-fpm.pid
  error_log = /usr/local/php-fpm/var/log/php-fpm.log
  [www]
  #监听地址，也可以写为127.0.0.1:9000
  listen = /tmp/php-fcgi.sock	
  #定义socket文件的权限
  listen.mode = 666
  #定义服务启动的用户和用户组
  user = php-fpm
  group = php-fpm
  #后面是进程配置
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  rlimit_files = 1024
  ```

- 之后从源码包中复制启动脚本并增加执行权限，设置为自动启动：

  ```bash
  [root@evobot etc]# cp /usr/local/src/php-5.6.32/sapi/fpm/init.d.php-fpm /etc/init.d/php-fpm
  
  [root@evobot etc]# chmod 755 /etc/init.d/php-fpm
  [root@evobot etc]# chkconfig --add php-fpm
  [root@evobot etc]# chkconfig php-fpm on
  ```

- 配置完成后，尝试启动php-fpm:

  ```bash
  [root@evobot etc]# service php-fpm start
  Starting php-fpm  done
  
  [root@evobot etc]# ps aux | grep php-fpm
  root     11728  0.0  0.2 123652  4920 ?        Ss   23:32   0:00 php-fpm: master process (/usr/local/php-fpm/etc/php-fpm.conf)
  php-fpm  11729  0.0  0.2 125736  5032 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11730  0.0  0.2 125736  5032 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11731  0.0  0.2 125736  5032 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11732  0.0  0.2 125736  5032 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11733  0.0  0.2 125736  5036 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11734  0.0  0.2 125736  5036 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11735  0.0  0.2 125736  5036 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11736  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11737  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11738  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11739  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11740  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11741  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11742  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11743  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11744  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11745  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11746  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11747  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  php-fpm  11748  0.0  0.2 125736  5040 ?        S    23:32   0:00 php-fpm: pool www
  root     11786  0.0  0.0 112676   984 pts/1    R+   23:33   0:00 grep --color=auto php-fpm
  
  [root@evobot etc]# ls -l /tmp/php-fcgi.sock 
  srw-rw-rw- 1 root root 0 6月   6 23:32 /tmp/php-fcgi.sock
  ```

  > 可以看到进程中启动的是www这个配置模块，并且socket文件的权限为666

# Nginx介绍

- Nginx可以提供web服务、反向代理、以及负载均衡；
- Nginx比较著名的分支，是淘宝基于Nginx开发的Tengine，使用上与Nginx一致，服务名、配置文件名都相同，但增加了一些定制化模块，在安全限速方面比较突出，同时支持对js，css合并；
- Nginx核心+lua相关的组件和模块组成了支持lua的高性能web容器openresty。

