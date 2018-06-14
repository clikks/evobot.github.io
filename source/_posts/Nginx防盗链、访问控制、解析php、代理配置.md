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
    fastcgi_param SCRIPT_FILENAME /data/wwwroot/auth.evobot.cn$fastcgi_script_name;
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

# Nginx常见502错误

1. 配置错误。一般是`fastcgi_pass`后面的php-fpm的pool的socket路径配置错误，或者是ip:port错误；

2. 资源耗尽。

  - LNMP架构在处理php时，nginx直接调取后端php-fpm服务，如果nginx请求量偏高，而php-fpm又没有配置足够的子进程，那么php-fpm就会资源耗尽，一旦资源好近nginx找不到php-fpm就会出现502错误；

  - 解决这种问题需要调整php-fpm.conf中的`pm.max_children`数值，使其增加，一般4G内存的机器，如果运行php-fpm和nginx,不运行mysql，可以设置为150，8G内存设置为300，以此类推；

3. 使用socket方式的php-fpm，默认监听的socket文件权限为所有者只读，属組和其他用户没有任何权限，所以nginx的启动用户(配置的为nobody)就无法读取这个文件，所以导致502错误，这个问题可以在nginx的错误日志中发现；

  - nginx的错误日志位于`/usr/local/nginx/logs/nginx_error.log`，在`/usr/local/nginx/conf/nginx.conf`中可以配置error_log，默认为crit，可以改为debug显示更全面的信息，但日志所占磁盘空间也会非常大；
 
  - 解决这种权限错误的问题，可以在php-fpm的pool配置中，增加`listen.owner`和`listen.group`，配置如下：
  
  ```bash
  [www]
  listen = /tmp/www.sock
  user = php-fpm
  group = php-fpm
  listen.owner = nobody    //定义属主
  listen.group = nobody    //定义属组
  pm = dynamic
  pm.max_children = 50
  pm.start_servers = 20
  pm.min_spare_servers = 5
  pm.max_spare_servers = 35
  pm.max_requests = 500
  rlimit_files = 1024
  ```
  > 配置完成后重启php-fpm即可，除了这种设置，还可以增加`listen.mode=666`这样的配置也可以解决问题。
  
# Nginx的location优先级

- 在nginx配置文件中，location按照优先级排序主要有以下几种形式：
  
  - 精确匹配`location = /abc {}`；
  - 匹配路径的前缀，如果找到就停止搜索`location ^~ /abc {}`；
  - 不区分大小写的正则匹配`location ~* /abc {}`；
  - 正则匹配`location ~ /abc {}`；
  - 普通路径前缀匹配`location /abc {}`;
  
- 各个格式的配置如下：

  ```bash
  # 精确匹配 / ,主机名后面不能带任何字符串
  location = /
  {
    [configuration A]
  }
  
  # 地址以 / 开头，下面的规则会匹配所有请求，但正则和最长字符串会优先匹配
  location / 
  {
    [configuration B]
  }
  
  # 匹配任何以/documents/开头的地址，匹配符合以后，还要继续向下搜索
  # 只有后面的正则表达式没有匹配时，这一条配置才会采用
  location /documents/
  {
    [configuration C]
  }
  
  location ~ /documents/ 
  {
    [configuration CB]
  }
  
  # 匹配任何以/documents/开头的地址，匹配符合以后，还要继续向下搜索
  # 只有后面的正则表达式没有匹配时，这一条配置才会采用
  location ~ /documents/Abc
  {
    [configuration CC]
  }
  
  # 匹配任何以/images/开头的地址，匹配符合以后，停止往下搜索正则并采用本条配置
  location ^~/iamges/
  {
    [configuration D]
  }
  
  # 匹配所有以gif,jpg或jpeg结尾的请求
  # 但所以请求/images/下的图片会被上面的configuration D处理，因为^~ 匹配后到达不了这条正则
  location ~* \.(gif|jpg|jpeg)$
  {
    [configuration E]
  }
  
  # 字符匹配到/images/，继续往下会发现^~存在
  location /images/ 
  {
    [configuration F]
  }
  
  # 最长字符匹配到/images/abc，继续往下会发现^~存在
  # F与G的放置顺序没有关系
  location /images/abc
  {
    [configuration G]
  }
  
  # 只有去掉configuration D才有效，先最长匹配configuration G开头的地址，继续向下搜索，匹配到这一条正则并采用
  location ~ /images/abc/
  {
    [configuration H]
  }
  ```
  
- 对A-H配置执行的顺序分析如下：

  1. 下面2个配置同时存在时：
  
  ```bash
  location = / 
  {
    [configuration A]
  }

  location / 
  {
    [configuration B]
  }
  ```
  - 此时A生效，因为`=/`优先级高于`/`；
  
  2. 下面3个配置同时存在时：
  
  ```bash
  location  /documents/ 
  {
    [configuration C]
  }

  location ~ /documents/ 
  {
    [configuration CB]
  }

  location ~ /documents/abc 
  {
    [configuration CC]
  }
  ```
  
  - 当访问的url为/documents/abc/1.html，此时CC生效，首先CB优先级高于C，而CC更优先于CB，因为CC更加精准；
  
  3. 下面4个配置同时存在时：
  
  ```bash
  location ^~ /images/
  {
    [configuration D]
  }

  location /images/ 
  {
    [configuration F]
  }

  location /images/abc
  {
    [configuration G ]
  }

  location ~ /images/abc/
  {
    [configuration H ]
  }
  ```
  
  - 当访问的链接为/images/abc/123.jpg时，此时D生效。虽然4个规则都能匹配到，但`^~`优先级是最高的。若`^~`不存在时，H优先，因为`~/images/ > /images/`，而/images/和/images/abc同时存在时，/images/abc优先级更高，因为后者更加精准

  4. 下面两个配置同时存在时：
  
  ```bash
  location ~* \.(gif|jpg|jpeg)$
  {
    [configuration E]
  }

  location ~ /images/abc/ 
  {
    [configuration H]
  }
  ```
  
  - 当访问的链接为/images/abc/123.jpg时，E生效。因为上面的规则更加精准。
---

