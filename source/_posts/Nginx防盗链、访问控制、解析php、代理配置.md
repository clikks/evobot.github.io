---
title: Nginx防盗链、访问控制、解析php、代理配置
author: Evobot
date: 2018-06-10 22:53:03
categories: LNMP
tags: [Linux, Centos, Nginx]
image:
---



本文主要介绍如何配置Nginx防盗链、访问控制，如何配置解析php，以及使用Nginx代理配置。

<!--more-->

---

# Nginx防盗链配置

- 在需要配置防盗链的Nginx的虚拟主机配置文件中，加入以下配置内容：

  ```bash
  location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip|doc|pdf|gz|bz2|jpeg|bmp|xls)$
  {
      expires 7d;
      valid_referers none blocked server_names  *.evobot.com;
      if ($invalid_referer) {
          return 403;
      }
      access_log off;
  }
  ```

  > 这部分配置包含了静态文件缓存过期时间配置以及不记录静态文件访问日志，并且location的`~*`表示正则匹配不区分大小写，其中`valid_referers`定义防盗链的白名单referer，

- 配置完成后的虚拟主机配置文件如下：

  ```bash
  server
  {
      listen 80;
      server_name auth.evobot.cn test.evobot.cn;
      index index.html index.htm index.php;
      root /data/wwwroot/auth.evobot.cn;
      if ($host != 'auth.evobot.cn') {
          rewrite ^/(.*)$ http://auth.evobot.cn/$1 permanent;
      }
      access_log /tmp/access_auth.log combined_realip;
  
      location ~ admin.html
      {
          auth_basic              "Auth";
          auth_basic_user_file    /usr/local/nginx/conf/htpasswd;
      }
  
      #location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
      #{
      #    expires    7d;
      #    access_log off;
      #}
  
      location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip|doc|pdf|gz|bz2|jpeg|bmp|xls)$
      {
          expires 7d;
          valid_referers none blocked server_names *.evobot.com;
          if ($invalid_referer) {
              return 403;
          }
          access_log off;
      }
  
      location ~ .*\.(js|css)$
      {
          expires    12h;
          access_log off;
      }
  }
  ```

- 重新加载配置文件后，使用curl测试：

  ```bash
  [root@evobot nginx]# /usr/local/nginx/sbin/nginx -t
  nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
  [root@evobot nginx]# service nginx reload
  Reloading nginx configuration (via systemctl):             [  确定  ]
  
  #直接访问，空referer访问成功
  [root@evobot nginx]# curl -x118.24.153.130:80 -I auth.evobot.cn/red.jpg
  HTTP/1.1 200 OK
  Server: nginx/1.12.2
  Date: Sun, 10 Jun 2018 15:07:28 GMT
  Content-Type: image/jpeg
  Content-Length: 1697128
  Last-Modified: Fri, 08 Jun 2018 19:01:27 GMT
  Connection: keep-alive
  ETag: "5b1ad287-19e568"
  Expires: Sun, 17 Jun 2018 15:07:28 GMT
  Cache-Control: max-age=604800
  Accept-Ranges: bytes
  
  #定义referer，访问报403状态码
  [root@evobot nginx]# curl -e "http://www.baidu.com" -x118.24.153.130:80 -I auth.evobot.cn/red.jpg
  HTTP/1.1 403 Forbidden
  Server: nginx/1.12.2
  Date: Sun, 10 Jun 2018 15:07:59 GMT
  Content-Type: text/html
  Content-Length: 169
  Connection: keep-alive
  
  #referer为白名单域名，访问成功
  [root@evobot nginx]# curl -e "http://www.evobot.com" -x118.24.153.130:80 -I auth.evobot.cn/red.jpg
  HTTP/1.1 200 OK
  Server: nginx/1.12.2
  Date: Sun, 10 Jun 2018 15:08:06 GMT
  Content-Type: image/jpeg
  Content-Length: 1697128
  Last-Modified: Fri, 08 Jun 2018 19:01:27 GMT
  Connection: keep-alive
  ETag: "5b1ad287-19e568"
  Expires: Sun, 17 Jun 2018 15:08:06 GMT
  Cache-Control: max-age=604800
  Accept-Ranges: bytes
  ```

# 访问控制

## 目录访问控制

- Nginx同样可以实现访问控制，如控制/admin/目录的请求只允许指定的IP访问，配置如下：

  ```bash
  location /admin/
  {
      allow 118.24.153.130;
      allow 127.0.0.1;
      allow 122.227.159.134
      deny all;
  }
  ```

  > 定义访问控制需要以先allow后deny的顺序进行配置。并且nginx在匹配到一个条件后，后面的条件就不会再进行匹配，如匹配到allow 127.0.0.1，后面的allow和deny都不会再执行。

- 配置完成后的虚拟主机配置文件如下:

  ```bash
  server
  {
      listen 80;
      server_name auth.evobot.cn test.evobot.cn;
      index index.html index.htm index.php;
      root /data/wwwroot/auth.evobot.cn;
      if ($host != 'auth.evobot.cn') {
      rewrite ^/(.*)$ http://auth.evobot.cn/$1 permanent;
      }
      access_log /tmp/access_auth.log combined_realip;
  
      location /admin/
      {
          allow 118.24.153.130;
          allow 127.0.0.1;
          allow 122.227.159.134;
          deny all;
      }
  
      location ~ admin.html
      {
          auth_basic      "Auth";
          auth_basic_user_file    /usr/local/nginx/conf/htpasswd;
      }
  
      location ~* ^.+\.(gif|jpg|png|swf|flv|rar|zip|doc|pdf|gz|bz2|jpeg|bmp|xls)$
      {
          expires 7d;
          valid_referers none blocked server_names *.evobot.com;
          if ($invalid_referer) {
              return 403;
      }
      access_log off;
      }
  
      location ~ .*\.(js|css)$
      {
          expires    12h;
          access_log off;
      }
  }
  ```

- 重新加载配置后，对/admin/进行访问：

  ```bash
  #从服务器本地访问
  [root@evobot nginx]# curl -x118.24.153.130:80 -I auth.evobot.cn/admin/index.html
  HTTP/1.1 200 OK
  Server: nginx/1.12.2
  Date: Sun, 10 Jun 2018 15:40:34 GMT
  Content-Type: text/html
  Content-Length: 22
  Last-Modified: Thu, 07 Jun 2018 18:23:59 GMT
  Connection: keep-alive
  ETag: "5b19783f-16"
  Accept-Ranges: bytes
  
  #从其他IP地址访问
  $ curl -x118.24.153.130:80 -I auth.evobot.cn/admin/index.html
  HTTP/1.1 403 Forbidden
  Server: nginx/1.12.2
  Date: Sun, 10 Jun 2018 15:51:24 GMT
  Content-Type: text/html
  Content-Length: 169
  Connection: keep-alive
  
  #查看访问日志
  118.24.153.130 - [10/Jun/2018:23:40:34 +0800] auth.evobot.cn "/admin/index.html" 200 "-" "curl/7.29.0"
  118.113.33.93 - [10/Jun/2018:23:51:24 +0800] auth.evobot.cn "/admin/index.html" 403 "-" "curl/7.47.0"
  ```

## 正则匹配访问控制

- 除了针对目录进行访问控制以外，还可以使用正则对URL进行匹配，对匹配到的访问进行控制；

- 例如禁止访问upload或者image目录下的php文件，在虚拟主机配置中添加以下配置：

  ```bash
  localtion ~ .*(upload|image)/.*\.php$
  {
      deny all;
  }
  ```

- 添加完成后，为虚拟主机创建相应的文件和目录，重新加载配置，然后尝试进行访问：

  ```bash
  [root@evobot nginx]# curl -x118.24.153.130:80 auth.evobot.cn/upload/test.php -I
  HTTP/1.1 403 Forbidden
  Server: nginx/1.12.2
  Date: Sun, 10 Jun 2018 16:52:58 GMT
  Content-Type: text/html
  Content-Length: 169
  Connection: keep-alive
  
  [root@evobot nginx]# curl -x118.24.153.130:80 auth.evobot.cn/upload/test.html -I
  HTTP/1.1 200 OK
  Server: nginx/1.12.2
  Date: Sun, 10 Jun 2018 16:54:06 GMT
  Content-Type: text/html
  Content-Length: 11
  Last-Modified: Sun, 10 Jun 2018 16:53:51 GMT
  Connection: keep-alive
  ETag: "5b1d579f-b"
  Accept-Ranges: bytes
  ```

## user_agent访问控制

- 根据user_agent进行访问控制，可以添加以下配置：

  ```bash
  if ($http_user_agent ~* 'Spider/3.0|YoudaoBot|Tomato')
  {
      return 403;
  }
  ```

- 重新加载配置后，进行访问：

  ```bash
  [root@evobot nginx]# curl -A "YOUDAObot" -x118.24.153.130:80 auth.evobot.cn/index.html -I
  HTTP/1.1 403 Forbidden
  Server: nginx/1.12.2
  Date: Sun, 10 Jun 2018 16:59:52 GMT
  Content-Type: text/html
  Content-Length: 169
  Connection: keep-alive
  ```


## Nginx解析PHP

- Nginx要解析PHP需要在虚拟主机配置文件中加入以下配置：

  ```bash
  location ~ \.php$
  {
      include fastcgi_params;
      fastcgi_pass unix:/tmp/php-fcgi.sock;
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME /data/wwwroot/auth.evobot.cn$fastcgi_script
  _name;
      }
  ```

  > 其中`fastcgi_pass`表示php-fpm的socket地址，在**php-fpm.conf**中定义，如果这里socket地址错误，则会报502错误，并且在nginx_error.log显示如下信息：
  >
  > ```bash
  > [root@evobot conf]# cat ../logs/nginx_error.log
  > 2018/06/11 23:54:12 [crit] 8871#0: *1 connect() to unix:/tmp/php-fci.sock failed(2: No such file or directory) while connecting to upstream, client: 127.0.0.1, server: auth.evobot.cn, request: "HEAD HTTP://auth.evobot.cn/index.php HTTP/1.1",upstream: "fastcgi://unix:/tmp/php-fci.sock:", host: "auth.evobot.cn"
  > ```

- 完成后，添加php页面，重新加载配置，尝试访问：

  ```bash
  [root@evobot conf]# curl -x127.0.0.1:80 auth.evobot.cn/index.php -I
  HTTP/1.1 200 OK
  Server: nginx/1.12.2
  Date: Mon, 11 Jun 2018 15:47:18 GMT
  Content-Type: text/html; charset=UTF-8
  Connection: keep-alive
  X-Powered-By: PHP/5.6.32
  ```

- 如果将php-fpm.conf里监听的socket改为IP地址，并重启php-fpm：

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
  ```

  > 如果使用socket形式时，这里的`listen.mode`没有配置，也会导致502错误，因为php-fcgi.sock的权限会变为440，nginx读取socket会报权限不足错误，因为nginx的用户为nobody。

- 然后还需要更改虚拟主机配置的`fastcgi_pass`：

  ```bash
  location ~ \.php$
  {
      include fastcgi_params;
      fastcgi_pass 127.0.0.1:9000
      fastcgi_index index.php;
      fastcgi_param SCRIPT_FILENAME /data/wwwroot/auth.evobot.cn$fastcgi_script_name;    }
  ```

- 重新加载配置后，同样可以解析php。

# Nginx代理

- Nginx代理的结构如下图：

  ![Nginx_proxy](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/Nginx_proxy.png)

- 配置Nginx代理，需要新建一个虚拟主机配置文件，如`proxy.conf`，然后写入如下配置：

  ```bash
  server
  {
      listen 80;
      server_name www.apelearn.com;
  
      location /
      {
          proxy_pass   http://47.104.7.242/;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP  $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }
  ```

  > server_name为代理的服务器的域名，proxy_pass则是被代理的服务器的IP地址，proxy_set_header则是设置访问的域名为server_name，最后两个则是指定X-Real-IP和X-Forwarded-For这两个变量的值。

- 然后使用curl测试如下：

  ```bash
  [root@evobot conf]# curl -x127.0.0.1:80 www.apelearn.com/ -I
  HTTP/1.1 200 OK
  Server: nginx/1.12.2
  Date: Mon, 11 Jun 2018 17:07:34 GMT
  Content-Type: text/html; charset=UTF-8
  Connection: keep-alive
  Vary: Accept-Encoding
  X-Powered-By: PHP/5.6.10
  ```

  > 这里就是访问本机但是实际上代理到了www.apelearn.com站点。

---

