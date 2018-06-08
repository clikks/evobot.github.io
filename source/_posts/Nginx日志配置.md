---
title: Nginx日志配置
author: Evobot
date: 2018-06-09 01:50:51
categories: LNMP
tags: [Linux, Centos, Nginx]
image:
---



本文主要介绍Nginx访问日志的一些相关配置，如配置日志切割和排除静态文件访问记录以及过期时间设置。

<!--more-->

---

# Nginx访问日志格式

- Nginx的日志记录格式实在Nginx的主配置文件中定义的，打开`nginx.conf`文件，搜索`log_format`字段：

  ```bash
  http
  {
      include mime.types;
      default_type application/octet-stream;
      server_names_hash_bucket_size 3526;
      server_names_hash_max_size 4096;
      //日志格式定义，combined_realip为日志格式的名字
      log_format combined_realip '$remote_addr $http_x_forwarded_for [$time_local]'
      ' $host "$request_uri" $status'
      ' "$http_referer" "$http_user_agent"';
  ```

- 日志的对应字段如下：

<style>
table th:first-of-type {
    width: 120px;
}
</style>

|          字段           |       含义       |
| :---------------------: | :--------------: |
|     `$remote_addr`      | 客户端IP(公网ip) |
| `$http_x_forwarded_for` |   代理服务器IP   |
|      `$time_local`      |  服务器本地时间  |
|         `$host`         | 访问主机名(域名) |
|     `$request_uri`      |  访问的url地址   |
|        `$status`        |      状态码      |
|     `$http_referer`     |     referer      |
|   `$http_user_agent`    |    user_agent    |

> Nginx的配置文件中，每一项配置都是以`;`结尾，在两个`;`之间的配置，不论是否换行，都看作为一行。

- 为虚拟主机配置访问日志，需要修改对应的虚拟主机配置文件，配置如下：

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
      #增加access_log配置，并指定日志路径以及日志格式名
      access_log /tmp/access_auth.log combined_relip;
  
      location ~ admin.html
      {
          auth_basic              "Auth";
          auth_basic_user_file    /usr/local/nginx/conf/htpasswd;
      }
  }
  ```

- 配置完成后，检查配置文件正确性并重新加载服务即可生效：

  ```bash
  [root@evobot nginx]# sbin/nginx -t
  nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
  
  [root@evobot nginx]# service nginx reload
  ```

- 然后访问站点、查看日志：

  ```bash
  [root@evobot nginx]# curl auth.evobot.cn/admin/index.html
  Welcome to admin page
  
  [root@evobot nginx]# cat /tmp/access_auth.log
  118.24.153.130 - [09/Jun/2018:02:25:02 +0800] auth.evobot.cn "/admin/index.php" 404 "-" "curl/7.29.0"
  118.24.153.130 - [09/Jun/2018:02:25:11 +0800] test.evobot.cn "/admin/index.php" 301 "-" "curl/7.29.0"
  118.24.153.130 - [09/Jun/2018:02:25:20 +0800] auth.evobot.cn "/admin/index.html" 200 "-" "curl/7.29.0"
  ```

# 日志切割

- Nginx的日志切割需要自定义shell脚本，创建`/usr/local/sbin/nginx_logrotate.sh`文件，写入下面的脚本内容:

  ```bash
  #!/bin/bash
  #生成前一天的日期
  d=`date -d "-1 day" +%Y%m%d`
  logdir="/tmp/"
  nginx_pid="/usr/local/nginx/logs/nginx.pid"
  cd $logdir
  for log in `ls *.log`
  do
      mv $log $log-$d
  done
  #重新加载以生成新日志
  /bin/kill -HUP `cat $nginx_pid`
  ```

- 然后使用`sh +x /usr/local/sbin/nginx_logrotate.sh`命令执行脚本查看脚本是否能正确运行，`-x`选项为输出脚本执行的具体过程：

  ```bash
  [root@evobot nginx]# sh -x /usr/local/sbin/nginx_log_rotate.sh
  ++ date -d '-1 day' +%Y%m%d
  + d=20180608
  + logdir=/tmp/
  + nginx_pid=/usr/local/nginx/logs/nginx.pid
  + cd /tmp/
  ++ ls access_auth.log
  + for log in '`ls *.log`'
  + mv access_auth.log access_auth.log-20180608
  ++ cat /usr/local/nginx/logs/nginx.pid
  + /bin/kill -HUP 18411
  
  [root@evobot nginx]# ls /tmp/
  access_auth.log
  access_auth.log-20180608
  ```

- 配置了日志切割之后，还需要定期将过期的日志删除，可以使用`find`命令来实现：

  ```bash
  find /tmp -name '*.log-*' -type f -mtime +7 -exec rm {} \;
  ```

- 将脚本和定时删除的命令添加到定时任务里即可：

  ```bash
  0 0 * * * /bin/bash /usr/local/sbin/nginx_logrotate &
  0 0 * * * find /tmp/ -name '*.log-*' -type f -mtime +7 -exec rm -f {}\;
  ```

# 静态文件不记录日志及过期时间

- 在虚拟主机配置文件中写入以下配置：

  ```bash
  location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
  {
      expires    7d;
      access_log off;
  }
  
  location ~ .*\.(js|css)$
  {
      expires    12h;
      access_log off;
  }
  ```

  > 其中使用`~`匹配以.gif ，.png和js、css等结尾的URL，expires则是定义文件的缓存过期时间，7d为7天，`access_log off`则是关闭访问日志记录。

- 在虚拟主机站点目录下上传图片，然后进行访问并查看日志：

  ```bash
  [root@evobot nginx]# curl auth.evobot.cn/red.jpg -I
  HTTP/1.1 200 OK
  Server: nginx/1.12.2
  Date: Fri, 08 Jun 2018 19:02:16 GMT
  Content-Type: image/jpeg
  Content-Length: 1697128
  Last-Modified: Fri, 08 Jun 2018 19:01:27 GMT
  Connection: keep-alive
  ETag: "5b1ad287-19e568"
  Expires: Fri, 15 Jun 2018 19:02:16 GMT
  Cache-Control: max-age=604800
  Accept-Ranges: bytes
  
  [root@evobot nginx]# cat /tmp/access_auth.log
  118.24.153.130 - [09/Jun/2018:03:02:42 +0800] auth.evobot.cn "/" 200 "-" "curl/7.29.0"
  118.24.153.130 - [09/Jun/2018:03:03:25 +0800] auth.evobot.cn "/" 200 "-" "curl/7.29.0"
  118.24.153.130 - [09/Jun/2018:03:03:33 +0800] auth.evobot.cn "/" 200 "-" "curl/7.29.0"
  ```

  > 可以看到，图片的访问并没有被记录到日志里。并且curl的访问中有Cache-Control行，显示了文件的缓存过期时间。