---
title: Apache访问日志配置及静态元素过期时间配置
author: Evobot
categories: LAMP
tags:
  - Linux
  - Centos
  - Apache
abbrlink: 8097d660
date: 2018-05-30 21:12:11
image:
---



本文主要介绍如何配置Apache的访问日志，使其不记录静态文件访问和如何配置访问日志的切割，另外介绍如何配置静态元素的过期时间。

<!--more-->

---

# 访问日志配置

## 不记录静态文件请求

- 由于网站页面上存在大量的图片，css等静态元素，如果对这些元素的访问都记录在日志中，会导致访问日志快速增加，所以可以在Apache中配置不记录这些静态文件的访问日志；

- 修改虚拟主机配置文件`httpd-vhosts.conf`，在需要配置不记录静态文件请求的虚拟主机配置的`ErrorLog "logs/xtears.com-error_log"`下添加以下内容：

  ```bash
  <VirtualHost *:80>
      DocumentRoot "/data/wwwroot/xtears.cn"
      ServerName xtears.cn
      ServerAlias www.abc.com www.123.com
      ErrorLog "logs/xtears.cn-error_log"
      SetEnvIf Request_URI ".*\.gif$" img
      SetEnvIf Request_URI ".*\.jpg$" img
      SetEnvIf Request_URI ".*\.png$" img
      SetEnvIf Request_URI ".*\.bmp$" img
      SetEnvIf Request_URI ".*\.swf$" img
      SetEnvIf Request_URI ".*\.js$" img
      SetEnvIf Request_URI ".*\.css$" img
      CustomLog "logs/xtears.cn-access_log" common env=!img
  </VirtualHost>
  ```

- 保存退出之后暂不加载配置文件，然后尝试访问图片资源：

  ```bash
  $ curl -x118.24.153.130:80 xtears.cn/x.jpg -I
  HTTP/1.1 404 Not Found
  Date: Wed, 30 May 2018 16:43:40 GMT
  Server: Apache/2.4.33 (Unix) PHP/5.6.32
  Content-Type: text/html; charset=iso-8859-1
  ```

  ```bash
  [root@evobot apache2.4]# tail -n 3 logs/xtears.cn-access_log 
  222.186.129.155 - - [31/May/2018:00:43:40 +0800] "HEAD / HTTP/1.1" 200 -
  118.113.205.248 - - [31/May/2018:00:43:40 +0800] "HEAD http://xtears.cn/x.jpg HTTP/1.1" 404 -
  ```

  - 可以看到没有刷新配置文件的情况下，日志会记录静态文件的访问请求。

- 重新加载配置文件，再访问图片资源：

  ```bash
  $ curl -x118.24.153.130:80 xtears.cn/xxx.jpg -I
  HTTP/1.1 404 Not Found
  Date: Wed, 30 May 2018 16:46:14 GMT
  Server: Apache/2.4.33 (Unix) PHP/5.6.32
  Content-Type: text/html; charset=iso-8859-1
  ```

  ```bash
  [root@evobot apache2.4]# tail logs/xtears.cn-access_log 
  222.186.129.155 - - [31/May/2018:00:44:40 +0800] "HEAD / HTTP/1.1" 200 -
  124.14.16.204 - - [31/May/2018:00:44:45 +0800] "HEAD / HTTP/1.1" 200 -
  113.31.27.249 - - [31/May/2018:00:45:02 +0800] "HEAD / HTTP/1.1" 200 -
  223.94.95.141 - - [31/May/2018:00:45:05 +0800] "HEAD / HTTP/1.1" 200 -
  222.88.91.50 - - [31/May/2018:00:45:07 +0800] "HEAD / HTTP/1.1" 200 -
  222.186.129.155 - - [31/May/2018:00:45:40 +0800] "HEAD / HTTP/1.1" 200 -
  124.14.16.204 - - [31/May/2018:00:45:46 +0800] "HEAD / HTTP/1.1" 200 -
  113.31.27.249 - - [31/May/2018:00:46:02 +0800] "HEAD / HTTP/1.1" 200 -
  223.94.95.141 - - [31/May/2018:00:46:05 +0800] "HEAD / HTTP/1.1" 200 -
  222.88.91.50 - - [31/May/2018:00:46:07 +0800] "HEAD / HTTP/1.1" 200 -
  ```

  - 可以看到，访问静态资源的日志已经不再记录。

## 访问日志切割

- 日志持续记录，会导致磁盘空间越来越大，所以需要对访问日志配置自动切割，并自动删除过期的日志文件；

- 配置访问日志切割，在虚拟主机配置文件中的虚拟主机配置的`CustomLog`行修改如下：

  ```bash
  <VirtualHost *:80>
      DocumentRoot "/data/wwwroot/xtears.cn"
      ServerName xtears.cn
      ServerAlias www.abc.com www.123.com
      ErrorLog "logs/xtears.cn-error_log"
      SetEnvIf Request_URI ".*\.gif$" img
      SetEnvIf Request_URI ".*\.jpg$" img
      SetEnvIf Request_URI ".*\.png$" img
      SetEnvIf Request_URI ".*\.bmp$" img
      SetEnvIf Request_URI ".*\.swf$" img
      SetEnvIf Request_URI ".*\.js$" img
      SetEnvIf Request_URI ".*\.css$" img
      CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/xtears.cn-access_%Y%m%d_log 86400" common env=!img
  </VirtualHost>
  ```

  - 其中`rotatelogs`是apache自带的日志切割工具，`-l`选项表示按照本地时间而不是UTC时间进行切割，然后日志名结尾的`%Y%m%d`表示日志名以时间结尾命名，86400则表示一天切割一次，即86400秒。

- 重加载配置文件后，进行几次访问请求，然后查看日志：

  ```bash
  [root@evobot apache2.4]# ls logs/
  access_log  xtears.cn-access_20180531.log  xtears.com-access_log
  ```

  - 可以看到生成了以日期为结尾的新的日志。

- 然后还需要配置定时任务，定期删除过期的日志文件。

- Apache的rotatelogs的语法为：`rotatelogs [-l] logfile [rotationtime [offset]] | [filesizeM]`，其中`-l`不加的话日志时间会和实际时间相差8小时，而logfile如果包含`%`，则会被视为strftime()格式的字符串，否则会加上以秒为单位的`.ssss`的后缀，时间都是表示新日志开始的时间，`rotationtime`则表示日志文件以秒为单位滚动的间隔时间，`offset`则是相对于UTC时间相缠得分钟数，如果省略，表示为0并且使用UTC时间，比如要指定与UTC时差为-5小时的当地时间，则参数应该为**-300**，单位为分钟；`filesizeM`是指以文件大小滚动，而不是按时间或时差滚动；

- 如下两个例子，分别以天分割和以小时分割：

  ```bash
  #按每天分割
  CustomLog "|bin/rotatelogs -l logs/access_%Y%m%d.log 86400" combined
  
  #按每小时分割
  CustomLog "|bin/rotatelogs -l logs/access_%Y%m%d%H.log 3600" combined
  ```

- 除了使用rotatelogs，apache还可以使用`cronolog`进行日志分割，同样以小时和以天分割日志，例子如下：

  ```bash
  #按每天分割
  CustomLog "|bin/cronolog logs/access_%Y%m%d.log" combined
  
  #按每小时分割
  CustomLog "|bin/cronolog logs/access_%Y%m%d%h.log" combined
  ```

## 日志记录代理及真实ip

- 默认配置中，log日志的格式为` LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined`，其中%h是访问者的ip地址，但是如果用户使用代理进行访问，那么这里的ip就会是代理机器的ip地址；

- 为了将用户的真实辑器ip记录下来，可以将日志格式改成如下格式：

  ```bash
   LogFormat "%h %{X-FORWARDED-FOR}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
  ```

## 只记录指定URI日志

- 如果需要apache只记录我们指定的URI的日志，如请求www.aaa.com/aa/...这样的地址才会记录日志，那么需要在httpd.conf或者相关的虚拟主机配置文件中添加如下内容：

  ```bash
  SetEnvIf Request_URI "^/aaa/.*" aaa-request
  CustomLog "|/usr/local/apache/bin/rotatelogs -l /usr/local/apache/lgs/aaa-access_%Y%m%d.log 86400" combined env=aaa-request
  ```

  > 配置原理与不记录静态文件日志相同。

## 记录客户端请求域名

- 这里的请求域名不是指我们虚拟主机配置的ServerName，而是指类似*.abc.com这样的泛解析形式；
- Apache的LogFormat的`%V`变量就可以记录用户请求的域名，更多关于LogFormat变量的设置信息，可以参考[Apache日志详解](http://blog.51cto.com/longlei/2095594)这篇博文。

# 静态元素过期时间

- 浏览器访问网站时，会将静态的文件缓存在本地电脑，这样下次访问时就不需要重新到服务器下载静态文件；

- 在Apache中可以配置静态文件的缓存时间，一旦配置的缓存时间过期，浏览器就会重新去服务器下载静态资源；

- 在Apache中配置静态资源过期时间，需要使用到`expires`模块，在虚拟主机配置中更改配置如下：

  ```bash
  <VirtualHost *:80>
      DocumentRoot "/data/wwwroot/xtears.cn"
      ServerName xtears.cn
      ServerAlias www.abc.com www.123.com
      <IfModule mod_expires.c>
          //打开过期时间开关
          ExpiresActive on
          //设置图片格式的静态文件过期时间，可以以天设置也可以以小时设置
          ExpiresByType image/gif "access plus 1 days"
          ExpiresByType image/jpeg "access plus 24 hours"
          ExpiresByType image/png "access plus 24 hours"
          //设置css、js等过期时间
          ExpiresByType text/css "now plus 2 hours"
          ExpiresByType application/x-javascript "now plus 2 hours"
          ExpiresByType application/javascript "now plus 2 hours"
          ExpiresByType application/x-shockwave-flash "now plus 2 hours"
          //设置默认的过期时间为不过期
          ExpiresDefault "now plus 0 min"
      </IfModule>
      ErrorLog "logs/xtears.cn-error_log"
      SetEnvIf Request_URI ".*\.gif$" img
      SetEnvIf Request_URI ".*\.jpg$" img
      SetEnvIf Request_URI ".*\.png$" img
      SetEnvIf Request_URI ".*\.bmp$" img
      SetEnvIf Request_URI ".*\.swf$" img
      SetEnvIf Request_URI ".*\.js$" img
      SetEnvIf Request_URI ".*\.css$" img
      CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/xtears.cn-access_%Y%m%d.log 86400" common env=!img
  </VirtualHost>
  ```

- 然后检查`expires`模块是否打开，未打开时需要修改`httpd.conf`文件取消注释模块：

  ```bash
  [root@evobot apache2.4]# vi conf/httpd.conf
  LoadModule expires_module modules/mod_expires.so
  [root@evobot apache2.4]# bin/apachectl -t
  Syntax OK
  [root@evobot apache2.4]# bin/apachectl graceful
  [root@evobot apache2.4]# bin/apachectl -M | grep expires
   expires_module (shared)
  ```

- 接着使用浏览器的开发工具或者curl测试访问静态文件：

  ```bash
  $ curl -x118.24.153.130:80 xtears.cn/red.jpg -I  
  HTTP/1.1 200 OK
  Date: Wed, 30 May 2018 17:52:40 GMT
  Server: Apache/2.4.33 (Unix) PHP/5.6.32
  Last-Modified: Sun, 10 Jul 2016 09:38:08 GMT
  ETag: "19e568-53744cb14e000"
  Accept-Ranges: bytes
  Content-Length: 1697128
  Cache-Control: max-age=86400
  Expires: Thu, 31 May 2018 17:52:40 GMT
  Content-Type: image/jpeg
  ```

  - 可以看到请求头中多了`Cache-Control`这一项并且在`Expires`中列出了过期时间。

---

