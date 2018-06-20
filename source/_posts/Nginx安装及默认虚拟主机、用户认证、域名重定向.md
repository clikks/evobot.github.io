---
title: Nginx安装及默认虚拟主机、用户认证、域名重定向
author: Evobot
categories: LNMP
tags:
  - Linux
  - Centos
  - Nginx
abbrlink: 1785e4e8
date: 2018-06-07 22:56:29
image:
---



本文主要介绍如何安装Nginx，以及安装完成配置Nginx默认虚拟主机，配置用户认证以及域名重定向。

<!--more-->

---

# Nginx安装

## 编译安装

- 首先下载Nginx-1.12版本源码包并解压，Nginx的版本包在这个地址都可以下载：[http://nginx.org/download/](http://nginx.org/download/)

  ```bash
  wget http://nginx.org/download/nginx-1.12.2.tar.gz
  
  tar zxvf nginx-1.12.2.tar.gz
  cd nginx-1.12.2/
  ```

- 编译安装：

  ```bash
   ./configure --prefix=/usr/local/nginx
  
  make && make install
  ```

  > nginx编译，需要使用什么模块，就要将对应的编译参数加上，这里暂时不添加模块，直接编译。

- 查看安装目录：

  ```bash
  [root@evobot nginx-1.12.2]# cd /usr/local/nginx/
  [root@evobot nginx]# ls
  conf  html  logs  sbin
  conf  html  logs  sbin
  
  #配置文件目录
  [root@evobot nginx]# ls conf/
  fastcgi.conf            koi-win             scgi_params
  fastcgi.conf.default    mime.types          scgi_params.default
  fastcgi_params          mime.types.default  uwsgi_params
  fastcgi_params.default  nginx.conf          uwsgi_params.default
  koi-utf                 nginx.conf.default  win-utf
  
  #样例网页目录
  [root@evobot nginx]# ls html/
  50x.html  index.html
  
  #核心执行文件
  [root@evobot nginx]# ls sbin/
  nginx
  ```

  > nginx执行文件同样支持-t选项检查配置文件正确性：
  >
  > ```bash
  > [root@evobot nginx]# sbin/nginx -t
  > nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  > nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
  > ```

## 配置启动脚本

- 在`/etc/init.d/`目录下创建`nginx`为名的启动脚本文件，写入以下内容：

  ```bash
  #!/bin/bash
  # chkconfig: - 30 21
  # description: http service.
  # Source Function Library
  . /etc/init.d/functions
  # Nginx Settings
  NGINX_SBIN="/usr/local/nginx/sbin/nginx"
  NGINX_CONF="/usr/local/nginx/conf/nginx.conf"
  NGINX_PID="/usr/local/nginx/logs/nginx.pid"
  RETVAL=0
  prog="Nginx"
  start() 
  {
      echo -n $"Starting $prog: "
      mkdir -p /dev/shm/nginx_temp
      daemon $NGINX_SBIN -c $NGINX_CONF
      RETVAL=$?
      echo
      return $RETVAL
  }
  stop() 
  {
      echo -n $"Stopping $prog: "
      killproc -p $NGINX_PID $NGINX_SBIN -TERM
      rm -rf /dev/shm/nginx_temp
      RETVAL=$?
      echo
      return $RETVAL
  }
  reload()
  {
      echo -n $"Reloading $prog: "
      killproc -p $NGINX_PID $NGINX_SBIN -HUP
      RETVAL=$?
      echo
      return $RETVAL
  }
  restart()
  {
      stop
      start
  }
  configtest()
  {
      $NGINX_SBIN -c $NGINX_CONF -t
      return 0
  }
  case "$1" in
    start)
          start
          ;;
    stop)
          stop
          ;;
    reload)
          reload
          ;;
    restart)
          restart
          ;;
    configtest)
          configtest
          ;;
    *)
          echo $"Usage: $0 {start|stop|reload|restart|configtest}"
          RETVAL=1
  esac
  exit $RETVAL
  ```

- 然后更改文件权限：

  ```bash
  chmod 755 /etc/init.d/nginx
  ```

## 创建配置文件

- 默认Nginx的conf目录下有一个配置文件，但是这里我们将原有的配置文件备份，重新创建一份配置文件:

  ```bash
  mv conf/nginx.conf conf/nginx.conf.bak
  vi conf/nginx.conf
  ```

- 写入以下内容：

  ```bash
  user nobody nobody;
  worker_processes 2;
  error_log /usr/local/nginx/logs/nginx_error.log crit;
  pid /usr/local/nginx/logs/nginx.pid;
  worker_rlimit_nofile 51200;
  events
  {
      use epoll;
      worker_connections 6000;
  }
  http
  {
      include mime.types;
      default_type application/octet-stream;
      server_names_hash_bucket_size 3526;
      server_names_hash_max_size 4096;
      log_format combined_realip '$remote_addr $http_x_forwarded_for [$time_local]'
      ' $host "$request_uri" $status'
      ' "$http_referer" "$http_user_agent"';
      sendfile on;
      tcp_nopush on;
      keepalive_timeout 30;
      client_header_timeout 3m;
      client_body_timeout 3m;
      send_timeout 3m;
      connection_pool_size 256;
      client_header_buffer_size 1k;
      large_client_header_buffers 8 4k;
      request_pool_size 4k;
      output_buffers 4 32k;
      postpone_output 1460;
      client_max_body_size 10m;
      client_body_buffer_size 256k;
      client_body_temp_path /usr/local/nginx/client_body_temp;
      proxy_temp_path /usr/local/nginx/proxy_temp;
      fastcgi_temp_path /usr/local/nginx/fastcgi_temp;
      fastcgi_intercept_errors on;
      tcp_nodelay on;
      gzip on;
      gzip_min_length 1k;
      gzip_buffers 4 8k;
      gzip_comp_level 5;
      gzip_http_version 1.1;
      gzip_types text/plain application/x-javascript text/css text/htm 
      application/xml;
      server
      {
          listen 80;
          server_name localhost;
          index index.html index.htm index.php;
          root /usr/local/nginx/html;
          location ~ \.php$ 
          {
              include fastcgi_params;
              fastcgi_pass unix:/tmp/php-fcgi.sock;
              fastcgi_index index.php;
              fastcgi_param SCRIPT_FILENAME /usr/local/nginx/html$fastcgi_script_name;
          }    
      }
  }
  ```

- 其中部分配置文件的作用如下：

  ```bash
  #定义运行nginx服务的用户及用户组
  user nobody nobody;
  #定义最大子进程数量
  worker_processes 2;
  #定义错误日志
  error_log /usr/local/nginx/logs/nginx_error.log crit;
  #定义pid文件路径
  pid /usr/local/nginx/logs/nginx.pid;
  #定义最大打开文件数量
  worker_rlimit_nofile 51200;
  events
  {
      use epoll;	#使用epoll模式
      worker_connections 6000;	#最大连接数
  }
  #定义http相关的配置
  http
  {
    ...
    server  #对应于apache的虚拟主机，
      {
          listen 80;  #监听端口
          server_name localhost; #主机域名
          index index.html index.htm index.php;
          root /usr/local/nginx/html;  #网站根目录
          location ~ \.php$ #PHP解析配置
          {
              include fastcgi_params;
              fastcgi_pass unix:/tmp/php-fcgi.sock; #与php通信方式，可以为ip:port
              fastcgi_index index.php;
              fastcgi_param SCRIPT_FILENAME /usr/local/nginx/html$fastcgi_script_name;
          }    
      }
  }
  ```

## 启动nginx并解析PHP

- 配置完成后，使用`service nginx start`启动nginx，查看进程，可以看到一个父进程，两个子进程，即`worker_processes 2`:

  ```bash
  [root@evobot ~]# ps aux |grep nginx
  root     18411  0.0  0.0  20496   624 ?        Ss   01:15   0:00 nginx: masterprocess /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
  nobody   18415  0.0  0.1  25024  3508 ?        S    01:15   0:00 nginx: workerprocess
  nobody   18416  0.0  0.1  25024  3760 ?        S    01:15   0:00 nginx: workerprocess
  root     18497  0.0  0.0 112676   980 pts/4    R+   01:16   0:00 grep --color=auto nginx
  
  [root@evobot ~]# !net
  netstat -tlnpa
  Active Internet connections (servers and established)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State PID/Program name
  tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN 1/systemd
  tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN 18411/nginx: master
  ```

- 然后使用`curl localhost`即可访问到nginx的默认页面：

  ```html
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  <style>
      body {
          width: 35em;
          margin: 0 auto;
          font-family: Tahoma, Verdana, Arial, sans-serif;
      }
  </style>
  </head>
  <body>
  <h1>Welcome to nginx!</h1>
  <p>If you see this page, the nginx web server is successfully installed and
  working. Further configuration is required.</p>
  
  <p>For online documentation and support please refer to
  <a href="http://nginx.org/">nginx.org</a>.<br/>
  Commercial support is available at
  <a href="http://nginx.com/">nginx.com</a>.</p>
  
  <p><em>Thank you for using nginx.</em></p>
  </body>
  </html>
  ```

- 在nginx安装目录下的`html`目录下创建`1.php`，写入以下内容：

  ```php
  <?php
  echo "nginx php run success";
  ?>
  ```

- 然后使用`curl localhost/1.php`查看解析情况：

  ```bash
  [root@evobot nginx]# curl localhost/1.php
  nginx php run success
  ```

# Nginx默认虚拟主机

- 将nginx.conf中的server配置部分删除，并增加一行配置如下：

  ```bash
  include vhost/*.conf
  ```

- 然后再conf目录内创建`vhost`子目录，并创建evobot.conf：

  ```bash
  [root@evobot nginx]# mkdir conf/vhost
  [root@evobot nginx]# touch conf/vhost/evobot.conf
  ```

- 在`evobot.conf`写入如下内容：

  ```bash
  server
  {
      #default server表示指定为默认虚拟主机
      listen 80 default_server;
      server_name test.evobot.cn;
      index index.html index.htm index.php;
      root /data/wwwroot/default;
  }
  ```

- 然后创建/data/wwwroot/default目录，并创建index.php：

  ```bash
  [root@evobot nginx]# mkdir /data/wwwroot/defautl
  [root@evobot nginx]# vi /data/wwwroot/default/index.html
  [root@evobot nginx]# cat !$
  cat /data/wwwroot/default/index.html
  Welcome to default vhost
  ```

- 检查配置文件正确性后，使用`service nginx reload`重新加载配置：

  ```bash
  [root@evobot nginx]# /usr/local/nginx/sbin/nginx -t
  nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
  [root@evobot nginx]# service nginx reload
  Reloading nginx configuration (via systemctl):             [  确定  ]
  ```

- curl访问，查看结果：

  ```bash
  [root@evobot nginx]# curl localhost
  Welcome to default vhost
  
  [root@evobot nginx]# curl test.evobot.cn
  Welcome to default vhost
  ```

- **nginx的虚拟主机配置中如果没用default_server这句配置，那么nginx会将vhost目录中第一个配置作为默认虚拟主机，如a.conf。**

# nginx root&alias文件路径配置

- nginx指定文件路径的方式有`root`和`alias`两种方式，主要的区别在于nginx如何解释location后面的uri，这会使两者以不同的方式将请求映射到服务器文件上。

- root的配置如下：
  
  ```bash
  location ~ ^/weblogs/ {
    root /data/weblogs/www.test.com;
    autoindex on;
    auth_basic  "Restricted";
    auth_basic_user_file    passwd/weblogs;
  }
  ```
  - 如果一个请求的URI为`/weblogs/httplogs/www.test.com-access.log`，web服务器会返回服务器上`/data/weblogs/www.test.com/weblogs/httplogs/www.test.com-access.log`文件；
  - root会根据完整的URI请求来进行路径的映射，即映射的本地路径是包含了location的匹配内容部分的；
  
- alias的配置如下：
  
  ```bash
  location ^~ /binapp/ {
    limit_conn limit 4;
    limit_rate 200k;
    internal;
    alias /data/statics/bin/apps/;
  }
  ```
  - alias在映射时会将location后面的匹配内容路径丢弃，把当前匹配到的目录指向到指定的目录；
  - 如一个请求的URI为`/binapp/a.test.com/favicon.jpg`，web服务器将会映射到本地的`/data/statics/bin/apps/a.test.com/favicon.jpg`文件；
  - alias在配置时，目录名后面必须要加上`/`；并且alias只能在location块中配置。

# Nginx用户认证

## 站点用户认证

- 首先创建一个新的vhost来进行配置，如auth.conf，然后写入以下配置：

  ```bash
  server
  {
      listen 80;
      server_name auth.evobot.cn;
      index index.html index.htm index.php;
      root /data/wwwroot/auth.evobot.cn;
  
      location /
      {
          auth_basic              "Auth";
          auth_basic_user_file    /usr/local/nginx/conf/htpasswd;
      }
  }
  ```

  > location配置项内的配置就是用户认证的配置，auth_basic为用户认证的名称，auth_basic_user_file则是用户名密码文件。

- 然后使用apache的htpasswd命令来生成用户名密码文件，如果已经编译安装，则使用`/usr/local/apache/bin/htpasswd`命令即可，没有安装apache，则使用yum安装httpd软件包，生成密码文件命令如下：

  ```bash
  [root@evobot nginx]# /usr/local/apache2.4/bin/htpasswd -c /usr/local/nginx/conf/htpasswd evobot
  New password:
  Re-type new password:
  Adding password for user evobot
  
  [root@evobot nginx]# cat /usr/local/nginx/conf/htpasswd
  evobot:$apr1$GErMiBdC$i4ZJIMPTVzFxSmwbPp7n1/
  ```

  > 如果要添加多个用户，则第二次添加不需要使用htpasswd命令的`-c`选项，否则新的用户名密码会将之前的用户名密码覆盖。

- 将虚拟主机站点根目录及相关页面创建完成，检查配置文件正确性后重载nginx，然后访问虚拟主机：

  ```bash
  [root@evobot nginx]# curl auth.evobot.cn -I
  HTTP/1.1 401 Unauthorized
  Server: nginx/1.12.2
  Date: Thu, 07 Jun 2018 18:11:45 GMT
  Content-Type: text/html
  Content-Length: 195
  Connection: keep-alive
  WWW-Authenticate: Basic realm="Auth"
  
  [root@evobot nginx]# curl -uevobot:evobot auth.evobot.cn
  Congratulations!Correct passwd!
  ```

## 文件用户认证

- 不想针对整个虚拟主机站点进行用户认证，只想针对指定的文件访问请求进行认证，只需要在`location`配置后面加上指定的文件相对虚拟主机根目录的路径即可：

  ```bash
  server
  {
      listen 80;
      server_name auth.evobot.cn;
      index index.html index.htm index.php;
      root /data/wwwroot/auth.evobot.cn;
  
      location /admin/	#增加文件路径
      {
          auth_basic              "Auth";
          auth_basic_user_file    /usr/local/nginx/conf/htpasswd;
      }
  }
  ```

- 保存退出后重新加载配置，然后访问测试：

  ```bash
  [root@evobot nginx]# curl auth.evobot.cn
  Welcome to my website!
  
  [root@evobot nginx]# curl auth.evobot.cn/admin/
  <html>
  <head><title>401 Authorization Required</title></head>
  <body bgcolor="white">
  <center><h1>401 Authorization Required</h1></center>
  <hr><center>nginx/1.12.2</center>
  </body>
  </html>
  
  [root@evobot nginx]# curl -uevobot:evobot auth.evobot.cn/admin/
  Congratulations!Correct passwd!
  ```

## URL用户认证

- 针对访问一类URL进行用户认证，需要在`location`后面使用`~`进行匹配，配置如下：

  ```bash
  server
  {
      listen 80;
      server_name auth.evobot.cn;
      index index.html index.htm index.php;
      root /data/wwwroot/auth.evobot.cn;
  
      location ~ admin.html  #匹配包含admin.html的URL
      {
          auth_basic              "Auth";
          auth_basic_user_file    /usr/local/nginx/conf/htpasswd;
      }
  }
  ```

- 重加载后尝试访问：

  ```bash
  [root@evobot nginx]# curl auth.evobot.cn
  Welcome to my website!
  
  [root@evobot nginx]# curl auth.evobot.cn/admin/
  Welcome to admin page
  
  [root@evobot nginx]# curl auth.evobot.cn/admin/admin.html
  <html>
  <head><title>401 Authorization Required</title></head>
  <body bgcolor="white">
  <center><h1>401 Authorization Required</h1></center>
  <hr><center>nginx/1.12.2</center>
  </body>
  </html>
  
  [root@evobot nginx]# curl -uevobot:evobot auth.evobot.cn/admin/admin.html
  Congratulations!Correct passwd!
  ```

# Nginx域名重定向

- 更改虚拟主机配置文件，为`server_name`增加多个域名，然后使用`rewrite`实现跳转，配置如下：

  ```bash
  server
  {
      listen 80;
      #增加多个域名
      server_name auth.evobot.cn test.evobot.cn; 
      index index.html index.htm index.php;
      root /data/wwwroot/auth.evobot.cn;
      #配置域名跳转，$host是用户访问的域名，rewrite进行匹配并将域名后面的请求地址转发给跳转的域名，permanent为301跳转，302跳转则为redirect
      if ($host != 'auth.evobot.cn') {
          rewrite ^/(.*)$ http://auth.evobot.cn/$1 permanent;
      }
  
      location ~ admin.html
      {
          auth_basic              "Auth";
          auth_basic_user_file    /usr/local/nginx/conf/htpasswd;
      }
  }
  ```

- 配置完成后重新加载配置，进行尝试访问，可以看到Location已经跳转到指定的地址：

  ```bash
  [root@evobot nginx]# curl test.evobot.cn/admin/index.html -I
  HTTP/1.1 301 Moved Permanently
  Server: nginx/1.12.2
  Date: Thu, 07 Jun 2018 18:42:36 GMT
  Content-Type: text/html
  Content-Length: 185
  Connection: keep-alive
  Location: http://auth.evobot.cn/admin/index.html
  ```

> Nginx配置文件[nginx.conf详解](http://www.ha97.com/5194.html)。