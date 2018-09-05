---
title: redis慢查询、php扩展、存储session及主从配置
author: Evobot
categories: NoSQL
tags:
  - redis
abbrlink: 96d8d8aa
date: 2018-08-22 20:52:09
image:
---

1. redis慢查询日志 
2.  php安装redis扩展 
3. redis存储session 
4. redis主从配置 

<!--more-->

---

# redis慢查询日志

- mysql有慢查询日志，同样的redis也有慢查询日志；

- 编辑配置文件`/etc/redis.conf`，针对慢查询日志，可以设置两个参数，一个是执行时长，单位是微妙，另一个时慢查询日志的长度，当一个新的命令被写入日志时，最早的一条会从命令日志队列中被移除；

- 在配置文件中配置一下两个参数：

  ```bash
  slowlog-log-slower-than 1000 //单位ms，表示鳗鱼1000ms则记录日志
  
  slowlog-max-len 128 //定义日志长度，默认配置最多存储128条
  ```

- 我们可以将配置中的1000ms改小，在redis中演示慢查询日志：

  ```bash
  127.0.0.1:4192> keys *
  1) "setc"
  2) "hash_1"
  3) "zkey"
  4) "mykey"
  5) "setbb"
  127.0.0.1:4192> SLOWLOG get
  1) 1) (integer) 1 # 日志序号，从0开始
     2) (integer) 1534943116
     3) (integer) 22
     4) 1) "keys"	# 执行的命令
        2) "*"
     5) "127.0.0.1:55932"
     6) ""
  2) 1) (integer) 0
     2) (integer) 1534943111
     3) (integer) 305
     4) 1) "COMMAND"
     5) "127.0.0.1:55932"
     6) ""
  
  ```

- `SLOWLOG GET` 可以查看所有的慢查询日志，如果想看最近的几条日志，则使用`SLOWLOG GET [num]`,num指定输出日志的数量，redis会将最近的num条日志输出来，redis慢查询日志从0开始计数：

  ```bash
   127.0.0.1:4192> SLOWLOG get 1
  1) 1) (integer) 2
     2) (integer) 1534943123
     3) (integer) 25
     4) 1) "SLOWLOG"
        2) "get"
     5) "127.0.0.1:55932"
     6) ""
  127.0.0.1:4192> SLOWLOG get 2
  1) 1) (integer) 3
     2) (integer) 1534943278
     3) (integer) 18
     4) 1) "SLOWLOG"
        2) "get"
        3) "1"
     5) "127.0.0.1:55932"
     6) ""
  2) 1) (integer) 2
     2) (integer) 1534943123
     3) (integer) 25
     4) 1) "SLOWLOG"
        2) "get"
     5) "127.0.0.1:55932"
     6) ""
  
  ```

- `SLOWLOG LEN`可以查看慢日志的条数：

  ```bash
  127.0.0.1:4192> SLOWLOG len
  (integer) 1
  127.0.0.1:4192> keys *
  1) "zkey"
  2) "setc"
  3) "hash_1"
  4) "mykey"
  5) "setbb"
  127.0.0.1:4192> SLOWLOG len
  (integer) 2
  
  ```

# PHP安装redis模块

- 安装php的redis模块与安装memcached相同，首先下载php的redis模块安装包：[下载地址](http://pecl.php.net/get/redis-4.1.1.tgz)

- 解压模块包后，执行一下命令安装：

  ```bash
  $ /usr/local/php-fpm/bin/phpize
  
  $ ./configure --with-php-config=/usr/local/php-fpm/bin/php-config
  
  $ make && make install
  ```

- 安装成功后，编辑php.ini文件，添加`extension=redis.so`配置，然后确认模块是否生效：

  ```bash
  $ /usr/local/php-fpm/sbin/php-fpm -m |grep redis
  redis
  
  ```

- 最后重启php-fpm服务即可。

# redis存储session

- 在前面的文章中，我们实现了使用memcached存储session，现在我们来使用redis存储session；

- 在php-fpm的pool配置文件中，写入以下配置：

  ```bash
  php_value[session.save_handler] = redis
  php_value[session.save_path] = "tcp://127.0.0.1:4192"
  ```

- 然后在默认虚拟主机的目录中创建测试的php文件1.php：

  ```php
  <?php 
  session_start(); 
  if (!isset($_SESSION['TEST'])) { 
  $_SESSION['TEST'] = time(); 
  } 
  $_SESSION['TEST3'] = time(); 
  print $_SESSION['TEST']; 
  print "<br><br>"; 
  print $_SESSION['TEST3']; 
  print "<br><br>"; 
  print session_id(); 
  ?>
  ```

- 执行`curl localhost/1.php`：

  ```bash
  ]# curl localhost/1.php
  1534945973<br><br>1534945973<br><br>2ke2qlt3opped1q672c53airln
  
  ```

- 重复执行多次后，进入redis查看是否生成session：

  ```bash
  127.0.0.1:4192> keys *
  1) "PHPREDIS_SESSION:dgr1i9l9810an2s4s274pkn79c"
  2) "PHPREDIS_SESSION:2ke2qlt3opped1q672c53airln"
  3) "zkey"
  4) "setbb"
  5) "PHPREDIS_SESSION:40l7hgrim32l3g196n2bg4r9bo"
  6) "mykey"
  7) "setc"
  8) "PHPREDIS_SESSION:n81gi04rh0l3aa5b4nl8gootq8"
  9) "hash_1"
  127.0.0.1:4192> get PHPREDIS_SESSION:40l7hgrim32l3g196n2bg4r9bo
  "TEST|i:1534946050;TEST3|i:1534946050;"
  ```

- 如果使用php连接redis cluster，需要使用predis扩展模块，项目地址：[https://github.com/nrk/predis](https://github.com/nrk/predis)。

# redis主从配置

- 我们在一台主机上启动两个redis，模拟两台redis服务器，来演示redis主从；

- 首先复制redis配置文件，重命名后更改配置文件的port、dir、pidfile，logfile；

- 然后在从redis的配置文件中增加以下配置：

  ```bash
  slaveof 127.0.0.1 4192	//主redis的ip和端口
  masterauth this>is>passwd //如果主redis设置了密码，则需要配置此项
  ```

- 然后启动从redis服务：

  ```bash
  $ ps aux |grep redis
  root     24903  0.0  0.1 147312  2468 ?        Ssl  22:05   0:00 redis-server 127.0.0.1:4192
  root     24981  0.0  0.1 147312  2364 ?        Ssl  22:06   0:00 redis-server 127.0.0.1:4193
  
  $ netstat -tlnp|grep redis
  tcp        0      0 10.139.151.2:4192       0.0.0.0:*               LISTEN   24903/redis-server
  tcp        0      0 127.0.0.1:4192          0.0.0.0:*               LISTEN   24903/redis-server
  tcp        0      0 10.139.151.2:4193       0.0.0.0:*               LISTEN   24981/redis-server
  tcp        0      0 127.0.0.1:4193          0.0.0.0:*               LISTEN   24981/redis-server
  
  ```

- redis从服务器会自动同步数据：

  ```bash
  $ redis-cli -p 4193
  127.0.0.1:4193> keys *
  1) "zkey"
  2) "PHPREDIS_SESSION:n81gi04rh0l3aa5b4nl8gootq8"
  3) "setc"
  4) "PHPREDIS_SESSION:40l7hgrim32l3g196n2bg4r9bo"
  5) "setbb"
  6) "hash_1"
  7) "PHPREDIS_SESSION:2ke2qlt3opped1q672c53airln"
  8) "mykey"
  9) "PHPREDIS_SESSION:dgr1i9l9810an2s4s274pkn79c"
  
  ```

  - 可以看到，从redis上已经同步到了主redis的数据。

  ```bash
  127.0.0.1:4193> evobot get slaveof
  1) "slaveof"
  2) "127.0.0.1 4192"
  
  ```

- 在从redis的配置文件中还可以配置slave只读模式：

  ```bash
  slave-read-only yes
  ```

  ```bash
  127.0.0.1:4193> set abc 111
  (error) READONLY You can't write against a read only slave.
  
  ```

---


