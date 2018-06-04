---
title: PHP扩展模块安装
author: Evobot
date: 2018-06-04 20:33:47
categories: Centos7
tags: [Linux, Centos, LAMP]
image:
---



本文主要介绍如何为php安装扩展模块，以及php错误日志级别、php短标签、php.ini的详细介绍。

<!--more-->

---

# PHP安装扩展模块

- 当PHP已经编译安装完成，但是由于实际需求需要增加新的模块，那么就需要使用PHP动态扩展模块的安装方式进行安装；

- 例如安装redis模块，首先需要下载redis模块的源码包，然后重命名并解压缩：

  ```bash
  wget https://codeload.github.com/phpredis/phpredis/zip/develop
  mv develop phpredis-develop.zip
  unzip phpredis-develop.zip
  ```

- 然后进行的操作与普通的编译安装不同，由于源码包为php的扩展模块，所以需要进入源码包目录，使用`phpize`来生成configure文件：

  ```bash
  [root@evobot phpredis-develop]# /usr/local/php7/bin/phpize 
  Configuring for:
  PHP Api Version:         20160303
  Zend Module Api No:      20160303
  Zend Extension Api No:   320160303
  ```

  > 提示缺少autoconf，只需要yum安装即可。

- 执行了上面的命令后，目录内会生成一个configure文件：

  ```bash
  [root@evobot phpredis-develop]# ls -l configure
  -rwxr-xr-x 1 root root 451757 6月   4 22:01 configure
  ```

- 然后与普通源码包一样进行编译，一般需要使用`--with-php-config=/usr/local/php7/bin/php-config`选项即可：

  ```bash
  ./configure --with-php-config=/usr/local/php7/bin/php-config
  make && make install
  ```

- 执行了`make install`之后，会输出以下信息：

  ```bash
  [root@evobot phpredis-develop]# make install
  Installing shared extensions:     /usr/local/php7/lib/php/extensions/no-debug-zts-20160303/
  ```

- 编译完成的so文件就会放入上面的php扩展模块目录，但是执行` /usr/local/php7/bin/php -m | grep redis`并没有加载这个模块，需要编辑配置文件增加模块；

- 首先使用命令`/usr/local/php7/bin/php -i | grep -i  extension_dir`查看模块所在目录：

  ```bash
  [root@evobot phpredis-develop]# /usr/local/php7/bin/php -i | grep -i extension_dir
  extension_dir => /usr/local/php7/lib/php/extensions/no-debug-zts-20160303 => /usr/local/php7/lib/php/extensions/no-debug-zts-20160303
  sqlite3.extension_dir => no value => no value
  [root@evobot phpredis-develop]# ls /usr/local/php7/lib/php/extensions/no-debug-zts-20160303/
  opcache.so  redis.so
  ```

- 然后编辑php.ini文件，将`extension=redis.so`写入配置文件的extension配置后面即可生效：

  ```bash
  ;extension=php_xmlrpc.dll
  ;extension=php_xsl.dll
  extension=redis.so
  ```

  ```bash
  [root@evobot phpredis-develop]# /usr/local/php7/bin/php -m | grep redis
  redis
  ```

---

# php扩展配置

## PHP错误日志级别

- php的错误日志级别`error_reporting`一共有下面这些级别：

  - `E_ALL`：所有错误和警告（除E_STRICT外）；
  - `E_ERROR`：致命的错误，脚本的执行被暂停；
  - `E_RECOVERABLE_ERROR`：大多数的致命错误；
  - `E_WARNING`：非致命的运行时的错误，只警告，脚本执行不会停止；
  - `E_PARSE`：编译时解析错误，解析错误应该只由分析器生成；
  - `E_NOTICE`：脚本运行时产生的提醒（往往是脚本里的一些bug，如变量没有定义，此错误不会导致任务中断；
  - `E_STRICT`：脚本运行时产生的提醒信息，会包含php抛出的如何修改的建议信息；
  - `E_CORE_ERROR`：在php启动后发生的致命性错误；
  - `E_CORE_WARNING`：在php启动后发生的非致命性错误，即警告信息；
  - `E_COMPILE_ERROR`：php编译时产生的致命性错误；
  - `E_COMPILE_WARNING`：php编译时产生的警告信息；
  - `E_USER_ERROR`：用户生成的错误；
  - `E_USER_WARNING`：用户生成的告警；
  - `E_USER_NOTICE`：用户生成的提醒。

- 在指定日志级别时，可以使用多个上面的日志级别，连接符有`&`与、`~`非、`|`或三种，比如：

  ```bash
  errot_reporting=E_ALL & ~E_NOTICE
  ```

  > 表示错误级别为E_ALL且排除E_NOTICE。

## php开启短标签

- php.ini中有`short_open_tag`配置项，这是针对php语法的配置，如将选项设置为`Off`，那么php必须以下面的形式才能解析：

  ```php
  <?php
      phpinfo();
  ?>
  ```

- 如果配置为`short_open_tag=On`，那么php将能够解析下面形式的代码：

  ```php
  <?
      phpinfo();
  ?>
  ```

- php.ini配置文件的详细介绍，可以参考链接：[http://blog.51cto.com/legolas/493917](http://blog.51cto.com/legolas/493917)。

