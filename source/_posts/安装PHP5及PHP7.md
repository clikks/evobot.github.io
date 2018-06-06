---
title: 安装PHP5及PHP7
author: Evobot
categories: LAMP
tags:
  - Linux
  - Centos
  - PHP
abbrlink: 75d64810
date: 2018-05-28 09:23:45
image:
---



目前主流的PHP版本为5.6和7.1，LAMP中，PHP是作为httpd的一个模块进行加载的，这里主要介绍如何安装5.6和7.1的PHP。

<!--more-->

---

# 安装PHP5.6

## 编译安装

- 下载压缩包并解压：

  ```bash
   wget http://cn2.php.net/distributions/php-5.6.32.tar.bz2
   tar jxvf php-5.6.32.tar.bz2
  ```

- 编译安装：

  ```bash
  cd php-5.6.32/
  
  ./configure --prefix=/usr/local/php \
  --with-apxs2=/usr/local/apache2.4/bin/apxs \
  --with-config-file-path=/usr/local/php/etc \
  --with-mysql=/usr/local/mysql \
  --with-pdo-mysql=/usr/local/mysql \
  --with-mysqli=/usr/local/mysql/bin/mysql_config \
  --with-libxml-dir \
  --with-gd \
  --with-jpeg-dir \
  --with-png-dir \
  --with-freetype-dir \
  --with-iconv-dir \
  --with-zlib-dir \
  --with-bz2 \
  --with-openssl \
  --with-mcrypt \
  --enable-soap \
  --enable-gd-native-ttf \
  --enable-mbstring \
  --enable-sockets \
  --enable-exif
  
  make && make intstal
  ```

  - 编译参数中，`--with-apxs2`是apache的工具，能够自动配置模块加载，这也是将php放在最后安装的原因，让apache自动加载php模块，`--with-config-file-path`指定php的配置文件路径，php的配置文件为php.ini，`--with-mysql`为指定MySQL的路径，`--with-pdo-mysql`、`--with-mysqli`为mysql不同的驱动或库，因为需要让php能够读写mysql，要编译php支持mysql的函数，在php7中，不再需要`--with-mysql`。

## 报错处理

- 编译时报错如下，则需要安装`libxml2-devel`软件包：

  ```bash
  checking for xml2-config path...
  configure: error: xml2-config not found. Please check your libxml2 installation.
  ```

- 报错如下，需要安装`bzip2-devel`软件包：

  ```bash
  checking for BZip2 in default path... not found
  configure: error: Please reinstall the BZip2 distribution
  ```

- 报错如下，需要安装`libjpeg-devel`软件包：

  ```bash
  configure: error: jpeglib.h not found.
  ```

- 报错如下，需要安装`libpng-devel`软件包：

  ```bash
  configure: error: png.h not found.
  ```

- 报错如下，需要安装`freetype-devel`软件包：

  ```bash
  configure: error: freetype-config not found.
  ```

- 报错如下，需要安装`libmcrypt-devel`软件包：

  ```bash
  configure: error: mcrypt.h not found. Please reinstall libmcrypt.
  ```

## PHP相关文件

- php核心二进制文件为`/usr/local/php/bin/php`；

- 而在apache中的php模块则为`/usr/local/apache2.4/modules/libphp5.so`；

- 在apache中生成了php模块后，即使删除php的安装目录，也不会影响使用：

  ```bash
  [root@evobot php]# /usr/local/apache2.4/bin/apachectl -M
   alias_module (shared)
   php5_module (shared)
  ```

- php也可以查看加载的模块，并且这些模块均为静态加载：

  ```bash
  [root@evobot php]# bin/php -m
  [PHP Modules]
  bz2
  Core
  ctype
  date
  dom
  ereg
  exif
  fileinfo
  filter
  gd
  hash
  iconv
  json
  libxml
  mbstring
  mcrypt
  mysql
  mysqli
  openssl
  pcre
  PDO
  pdo_mysql
  pdo_sqlite
  Phar
  posix
  Reflection
  session
  SimpleXML
  soap
  sockets
  SPL
  sqlite3
  standard
  tokenizer
  xml
  xmlreader
  xmlwriter
  
  [Zend Modules]
  ```

- 查看apache的配置文件`/usr/local/apache2.4/conf/httpd.conf`，会有如下内容：

  ```bash
  #LoadModule actions_module modules/mod_actions.so
  #LoadModule speling_module modules/mod_speling.so
  #LoadModule userdir_module modules/mod_userdir.so
  LoadModule alias_module modules/mod_alias.so
  #LoadModule rewrite_module modules/mod_rewrite.so
  LoadModule php5_module        modules/libphp5.so
  ```

  - 这里注释的是不需要加载的模块，没有注释的就是使用时需要加载的模块以及模块路径。

- 最后，使用` /usr/local/php/bin/php -i | less`可以查看php的相关信息，如编译参数、配置文件路径，这里可以看到配置文件路径为`none`：

  ```bash
  System => Linux evobot 3.10.0-693.21.1.el7.x86_64 #1 SMP Wed Mar 7 19:03:37 UTC 2018 x86_64
  Build Date => May 28 2018 11:08:35
  Configure Command =>  './configure'  '--prefix=/usr/local/php' '--with-apxs2=/usr/local/apache2.4/bin/apxs' '--with-config-file-path=/usr/local/php/etc' '--with-mysql=/usr/local/mysql' '--with-pdo-mysql=/usr/local/mysql' '--with-mysqli=/usr/local/mysql/bin/mysql_config' '--with-libxml-dir' '--with-gd' '--with-jpeg-dir' '--with-png-dir' '--with-freetype-dir' '--with-icony-dir' '--with-zliv-dir' '--with-bz2' '--with-openssl' '--with-mcrypt' '--enable-soap' '--enable--gd-native-ttf' '--enable-mbstring' '--enable-sockets' '--enable-exif'
  Server API => Command Line Interface
  Virtual Directory Support => enabled
  Configuration File (php.ini) Path => /usr/local/php/etc
  Loaded Configuration File => (none)	#没有配置文件
  Scan this dir for additional .ini files => (none)
  Additional .ini files parsed => (none)
  PHP API => 20131106
  PHP Extension => 20131226
  Zend Extension => 220131226
  Zend Extension Build => API220131226,TS
  PHP Extension Build => API20131226,TS
  Debug Build => no
  
  ```

- 可以将源码包内的php的配置文件模板复制到配置文件路径，php.ini-production适合在生产环境使用，而development适合在开发环境使用：

  ```bash
  [root@evobot php-5.6.32] cp php.ini-production /usr/local/php/etc/php.ini
  ```

  

# 安装PHP7

## 编译安装

- [下载php-7.1.6源码包](http://cn2.php.net/distributions/php-7.1.6.tar.bz2)

- 解压缩后编译参数如下：

  ```bash
  ./configure --prefix=/usr/local/php7 \
  --with-apxs2=/usr/local/apache2.4/bin/apxs \
  --with-config-file-path=/usr/local/php7/etc \
  #--with-pdo-mysql=/usr/local/mysql \ //PHP7已更改为mysalnd
  --with-pdo-mysql=mysqlnd \
  #--with-mysqli=/usr/local/mysql/bin/mysql_config \ //PHP7已更改为mysalnd
  --with-mysqli=mysqlnd \
  --with-libxml-dir \
  --with-gd \
  --with-jpeg-dir \
  --with-png-dir \
  --with-freetype-dir \
  --with-iconv-dir \
  --with-zlib-dir \
  --with-bz2 \
  --with-openssl \
  --with-mcrypt \
  --enable-soap \
  --enable-gd-native-ttf \
  --enable-mbstring \
  --enable-sockets \
  --enable-exif
  ```

  ```bash
  make && make install
  ```

## 相关文件

- 安装完成后，在`/usr/local/apache2.4/modules/`会生成php7的模块，如果安装了多个php，在apache调用时，需要在apache的配置文件中将不需要使用的php注释掉。


