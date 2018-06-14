---
title: LNMP的php-fpm配置
author: Evobot
date: 2018-06-13 20:56:25
categories: LNMP
tags: [Linux, Centos, PHP]
image:
---



本文主要介绍LNMP的php-fpm的pool配置，以及php-fpm的慢执行日志，open_basedir、进程管理相关的知识。

<!--more-->

---

# php-fpm的pool

- 所谓的php-fpm的pool，实际上就是指我们在`/usr/local/php-fpm/etc/`下的php-fpm.conf配置文件中定义的`[www]`块的配置，整个[www]称为一个pool，实际上php-fpm支持定义多个pool，每个pool可以定义不同的pid，可以查看系统进程看到运行的pool：

  ```bash
  [root@evobot etc]# ps aux|grep php
  root       799  0.0  0.0 112676   984 pts/1    R+   21:10   0:00 grep --color=auto php
  root      9240  0.0  0.2 227224  4960 ?        Ss   6月11   0:06 php-fpm: master process (/usr/local/php-fpm/etc/php-fpm.conf)
  php-fpm   9241  0.0  0.2 229308  5044 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9242  0.0  0.2 229308  5044 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9243  0.0  0.2 229308  5044 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9244  0.0  0.2 229308  5044 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9245  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9246  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9247  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9248  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9249  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9250  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9251  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9252  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9253  0.0  0.3 229308  6736 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9254  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9255  0.0  0.3 229308  6740 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9256  0.0  0.2 229308  5048 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9257  0.0  0.2 229308  5052 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9258  0.0  0.2 229308  5052 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9259  0.0  0.2 229308  5052 ?        S    6月11   0:00 php-fpm: pool www
  php-fpm   9260  0.0  0.2 229308  5052 ?        S    6月11   0:00 php-fpm: pool www
  ```

- 为了防止一个站点出现问题，耗尽php资源，我们应该将每个站点进行隔离，使用单独的pool，在php-fpm.conf中，可以继续添加多个pool：

  ```bash
  [global]
  pid = /usr/local/php-fpm/var/run/php-fpm.pid
  error_log = /usr/local/php-fpm/var/log/php-fpm.log
  [www]
  #监听地址，也可以写为127.0.0.1:9000
  #listen = /tmp/php-fcgi.sock    
  listen = 127.0.0.1:9000
  #定义socket文件的权限
  listen.mode = 666
  #定义服务启动的用户和用户组
  user = php-fpm
  group = php-fpm
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  rlimit_files = 1024
  
  [evobot]
  listen = /tmp/php-evobot.sock
  listen.mode = 666
  user = php-fpm
  group = php-fpm
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  rlimit_files = 1024
  ```

- 然后检查配置文件正确性，重载服务，查看php-fpm进程：

  ```bash
  [root@evobot etc]# /usr/local/php-fpm/sbin/php-fpm -t
  [13-Jun-2018 21:19:52] NOTICE: configuration file /usr/local/php-fpm/etc/php-fpm.conf test is successful
  
  [root@evobot etc]# service php-fpm reload
  Reload service php-fpm  done
  [root@evobot etc]# ps aux | grep php
  root      1279  0.1  0.2 227284  4984 ?        Ss   21:20   0:00 php-fpm: master process (/usr/local/php-fpm/etc/php-fpm.conf)
  php-fpm   1282  0.0  0.2 229308  5060 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1283  0.0  0.2 229308  5060 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1284  0.0  0.2 229308  5060 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1285  0.0  0.2 229308  5060 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1286  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1287  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1288  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1289  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1290  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1291  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1292  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1293  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1294  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1295  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1296  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1297  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1298  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1299  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1300  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1301  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool www
  php-fpm   1302  0.0  0.2 229308  5060 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1303  0.0  0.2 229308  5060 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1304  0.0  0.2 229308  5060 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1305  0.0  0.2 229308  5060 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1306  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1307  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1308  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1309  0.0  0.2 229308  5064 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1310  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1311  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1313  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1314  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1315  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1316  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1317  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1318  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1319  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1320  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1321  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  php-fpm   1322  0.0  0.2 229308  5068 ?        S    21:20   0:00 php-fpm: pool evobot
  root      1329  0.0  0.0 112676   980 pts/1    R+   21:20   0:00 grep --color=auto php
  ```

  > 可以看到在进程中多了evobot的pool。

- 在Nginx中为不同的虚拟主机使用不同的php-fpm的pool，则需要修改虚拟主机配置文件，将其中的php配置的`fastcgi_pass`更改为指定的pool的socket路径：

  ```bash
   location ~ \.php$
      {
          include fastcgi_params;
          fastcgi_pass unix:/tmp/evobot.sock;
          fastcgi_index index.php;
          fastcgi_param SCRIPT_FILENAME /data/wwwroot/auth.evobot.cn$fastcgi_script_name;
      }
  ```

  > 这样对虚拟主机的php进行区分，可以防止一个虚拟主机访问量过大导致php资源耗尽报502错误时，不影响其他的虚拟主机运行。

- 在nginx.conf配置中，我们指定虚拟主机配置使用了`incluede vhost/*conf;`这样的配置来让nginx能够支持多个虚拟主机配置，同样的php-fpm也支持多个不同pool的配置文件，让不同的pool的配置独立到不同的配置文件中，这样的配置只需要在php-fpm.conf文件的`[global]`配置下，加入下面的配置：

  ```bash
  include = etc/php-fpm.d/*.conf
  ```

  ```bash
  [global]
  pid = /usr/local/php-fpm/var/run/php-fpm.pid
  error_log = /usr/local/php-fpm/var/log/php-fpm.log
  include = etc/php-fpm.d/*.conf
  ```

- 然后在`/usr/local/php-fpm/etc`目录下创建`php-fpm.d`目录，将php-fpm.conf配置文件中的两个pool分别写入到独立的配置文件中，并存放于php-fpm.d目录下：

  创建www.conf:

  ```bash
  [www]
  #监听地址，也可以写为127.0.0.1:9000
  #listen = /tmp/php-fcgi.sock    
  listen = 127.0.0.1:9000
  #定义socket文件的权限
  listen.mode = 666
  #定义服务启动的用户和用户组
  user = php-fpm
  group = php-fpm
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  rlimit_files = 1024
  ```

  创建evobot.conf:

  ```bash
  [evobot]
  listen = /tmp/php-evobot.sock
  listen.mode = 666
  user = php-fpm
  group = php-fpm
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  rlimit_files = 1024
  ```

- 然后重新检查配置文件正确性并重载服务，与之前的配置效果相同。

# php-fpm慢执行日志

- 有时候网站访问会比较慢，如果是系统负载引起的，我们可以使用各种命令行工具查看CPU、磁盘负载等等来找出导致负载高的进程，但如果是网站的代码导致的访问慢，如php代码层面的问题，那么配置慢执行日志，对我们排查问题将会非常有用；

- 慢执行日志需要在pool中配置以下配置项：

  ```bash
  request_slowlog_timeout = 1
  slowlog = /usr/local/php-fpm/var/log/www-slow.log
  ```

  配置完成后的pool配置如下：

  ```bash
  [www]
  listen = 127.0.0.1:9000
  listen.mode = 666
  user = php-fpm
  group = php-fpm
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  rlimit_files = 1024
  request_slowlog_timeout = 1
  slowlog = /usr/local/php-fpm/var/log/www-slow.log
  ```

- 重新加载配置和服务，可以在定义的slowlog路径中查看慢执行日志：

  ```bash
  [root@evobot etc]# cat /usr/local/php-fpm/var/log/www-slow.log 
  ```

- 由于没用执行慢于1秒的php，所以暂时日志里没有内容，测试慢日志，可以创建一个php文件，写入以下内容：

  ```php
  <?php
  echo "test slow log";
  sleep(2);
  echo "done";
  ?>
  ```

- 然后进行访问，如果出现错误，可以在php.ini中将`display_errors = Off`改为`On`，然后可以在浏览器中查看出错的原因。访问上面的php，然后查看慢日志如下：

  ```bash
  [root@evobot etc]# cat ../var/log/www-slow.log 
  
  [13-Jun-2018 22:19:41]  [pool www] pid 3992
  script_filename = /data/wwwroot/auth.evobot.cn/slow.php
  [0x00007f436d5826c8] sleep() /data/wwwroot/auth.evobot.cn/slow.php:3
  ```

  > 可以看到日志中记录了哪一条语句执行超过了1秒，导致了加载缓慢。一般实际使用中会将慢执行日志的配置设置为2秒。

# open_basedir

- 这里的open_basedir与LAMP中相同，都是为了防止站点出现安全问题时影响到其他的虚拟主机，我们可以针对每个pool定义其open_basedir，在pool的配置中，写入以下配置：

  ```bash
  php_admin_value[open_basedir]=/data/wwwroot/auth.evobot.cn:/tmp/
  ```

  pool的完整配置如下：

  ```bash
  [www]
  listen = 127.0.0.1:9000
  listen.mode = 666
  user = php-fpm
  group = php-fpm
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  rlimit_files = 1024
  request_slowlog_timeout = 1
  slowlog = /usr/local/php-fpm/var/log/www-slow.log
  php_admin_value[open_basedir]=/data/wwwroot/auth.evobot.cn:/tmp/
  ```

  > 这里php还需要访问tmp目录的socket，所以要加上tmp目录在配置中。

- 重新加载配置文件和重载php-fpm服务之后生效，如果open_basedir配置错误，访问站点会报错。

- 在php.ini中，配置错误日志，首先将`log_errors`置为On，然后将`error_reporting`设置为需要的日志级别，如`E_ALL`，最后配置`error_log`，指定日志存放路径：

  ```bash
  log_errors = On
  error_reporting = E_ALL
  error_log = /usr/local/php-fpm/var/log/php_errors.log
  ```

- 接着将虚拟主机使用的pool的open_basedir改为错误，进行访问并查看日志：

  ```bash
  php_admin_value[open_basedir]=/data/wwwroot/default:/tmp/
  
  ```

  ```bash
  [root@evobot etc]# curl -x127.0.0.1:80 auth.evobot.cn/index.php
  No input file specified.
  
  [root@evobot etc]# cat ../var/log/php_error.log
  [14-Jun-2018 09:11:26 UTC] PHP Warning:  Unknown: open_basedir restriction in effect. File(/data/wwwroot/test/index.php) is not within the allowed path(s): (/data/wwwroot/test2:/tmp/) in Unknown on line 0
  [14-Jun-2018 09:11:26 UTC] PHP Warning:  Unknown: failed to open stream: Operation not permitted in Unknown on line 0
  [14-Jun-2018 09:12:14 UTC] PHP Warning:  Unknown: open_basedir restriction in effect. File(/data/wwwroot/test/index.php) is not within the allowed path(s): (/data/wwwroot/test2:/tmp/) in Unknown on line 0
  [14-Jun-2018 09:12:14 UTC] PHP Warning:  Unknown: failed to open stream: Operation not permitted in Unknown on line 0
  ```
  > 如果出现日志无法记录的情况，要检查phpinfo中php.ini文件的位置，以及是否加载了php.ini配置文件。

# php-fpm进程管理

- 在pool的配置中，都有这样一段配置：

  ```bash
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  
  ```

- 这里`pm = dynamic`表示动态启动和销毁子进程，也可以定义为`static`，直接一次启动所有的子进程，并且`pm.max_children=50`下面的配置都不会再生效。

- `pm.max_children = 50`表示最大子进程数，php会在子进程数内动态启动、销毁子进程，但销毁子进程是有限度的，最低要保留5个子进程；

- `pm.start_servers = 20`则表示启动服务时会启动的进程数；

- `pm.min_spare_servers = 5`则定义了在空闲时段，子进程数的最少数量，如果达到这个数值，那么php-fpm服务会自动派生新的子进程；

- `pm.max_spare_servers = 35`定义了在空闲时段，子进程数的最大数量，如果高于这个数值，那么php-fpm就会开始清理空闲的子进程；

- `pm.max_requests = 500`则定义了一个子进程最多处理的请求数，即一个php-fpm的子进程可以处理这么多请求，当达到这个数值时，子进程就会自动退出。

---

