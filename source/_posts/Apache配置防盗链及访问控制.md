---
title: Apache配置防盗链及访问控制
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 7abd11c3
date: 2018-05-31 22:00:58
image:
---



本文主要介绍Apache如何配置防盗链，以及访问控制Directory和FilesMatch。

<!--more-->

---

# 防盗链配置

- 防盗链是为了防止外站引用本站的资源，导致本站流量上升，占用带宽资源；

- 防盗链的配置就是让资源只能在本站域名引用，不让其他域名引用，即`referer`，在虚拟主机配置里增加以下配置：

  ```bash
  <VirtualHost *:80>
      DocumentRoot "/data/wwwroot/xtears.cn"
      ServerName xtears.cn
      ServerAlias www.abc.com www.123.com
      <Directory /data/wwwroot/xtears.cn>
          SetEnvIfNoCase Referer "http://xtears.cn" local_ref
          SetEnvIfNoCase Referer "http://abc.com" local_ref
          SetEnvIfNoCase Referer "^$" local_ref
          <filesmatch "\.(txt|doc|mp3|zip|rar|jpg|gif)">
              Order Allow,Deny
              Allow from env=local_ref
          </filesmatch>
      </Directory>
      ErrorLog "logs/xtears.cn-error_log"
      CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/xtears.cn-access_%Y%m%d.log 86400" common 
  </VirtualHost>
  ```

  - 这里Directory设置了两个域名白名单：xtears.cn和abc.com，FilesMatch配置设置了需要做防盗链的文件格式，FilesMatch可以不区分大小写，然后在FilesMatch中配置了访问控制，让白名单的域名允许引用，其他的引用被Deny。
  - 在两个白名单域名后面有一个空referer：`SetEnvIfNoCase Referer "^$" local_ref`，空referer是指直接访问资源链接，并不是其他站点引用之后访问的，所以为了防止用户直接复制资源源地址访问被403拒绝，这里增加允许空referer的访问。

- 使用`curl -e "http://domain.com" -x127.0.0.1:80 xtears.cn/red.jpg -I`可以模拟referer，选项`-e`用来指定referer：

  ```bash
  $ curl -e "http://www.qq.com" -x118.24.153.130:80 xtears.cn/red.jpg -I
  HTTP/1.1 403 Forbidden
  Date: Thu, 31 May 2018 14:55:40 GMT
  Server: Apache/2.4.33 (Unix) PHP/5.6.32
  Content-Type: text/html; charset=iso-8859-1
  ```

  - 可以看到访问被403拒绝，而如果注释掉空Referer，直接访问也会返回403错误：

  ![403](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/403.png)

# 访问控制

## Directory

- 之前设置管理员的二次认证，在这个基础上，还可以进一步配置访问控制，只允许指定IP地址能够访问页面，Directory访问控制配置如下，写入虚拟主机配置：

  ```bash
  <VirtualHost *:80>
      DocumentRoot "/data/wwwroot/xtears.cn"
      ServerName xtears.cn
      ServerAlias www.abc.com www.123.com
      <Directory /data/wwwroot/xtears.cn/admin>
          Order deny,allow
          Deny from all
          Allow from 127.0.0.1
      </Directory>
  
      <Directory /data/wwwroot/xtears.cn>
          SetEnvIfNoCase Referer "http://xtears.cn" local_ref
          SetEnvIfNoCase Referer "http://abc.com" local_ref
          #SetEnvIfNoCase Referer "^$" local_ref
          <filesmatch "\.(txt|doc|mp3|zip|rar|jpg|gif)">
              Order Allow,Deny
              Allow from env=local_ref
          </filesmatch>
      </Directory>
      ErrorLog "logs/xtears.cn-error_log"
      CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/xtears.cn-access_%Y%m%d.log 86400" common
  </VirtualHost>
  ```

- 这里为了防止两个Directory冲突，所以将访问控制放在了上面，其中`Order`是定义访问控制的顺序，即先执行deny的语句还是allow的语句，控制语句不论是否匹配，都会按照Order的顺序执行，所以这里如果访问来自127.0.0.1，那么会先被deny拒绝，然后再被允许访问。

- 更新配置后，创建网站根目录下的admin目录和网页文件，然后在服务器本地尝试访问，结果被允许访问：

  ```bash
  # curl -x127.0.0.1:80 xtears.cn/admin/index.php -I
  HTTP/1.1 200 OK
  Date: Thu, 31 May 2018 15:14:54 GMT
  Server: Apache/2.4.33 (Unix) PHP/5.6.32
  X-Powered-By: PHP/5.6.32
  Content-Type: text/html; charset=UTF-8
  ```

- 从其他IP访问，返回403状态码：

  ```bash
  $ curl -x118.24.153.130:80 xtears.cn/admin/index.php -I
  HTTP/1.1 403 Forbidden
  Date: Thu, 31 May 2018 15:18:02 GMT
  Server: Apache/2.4.33 (Unix) PHP/5.6.32
  Content-Type: text/html; charset=iso-8859-1
  ```

## FilesMatch

- 上面的访问控制是控制目录的访问，另外也可以对链接或者文件进行访问控制，比如访问的链接后面带有参数的情况，这时就可以用FilesMatch来进行访问控制；

- 虚拟主机配置文件配置如下：

  ```bash
  <VirtualHost *:80>
      DocumentRoot "/data/wwwroot/xtears.cn"
      ServerName xtears.cn
      ServerAlias www.abc.com www.123.com
      <Directory /data/wwwroot/xtears.cn>
          <FilesMatch admin.php(.*)>
          Order deny,allow
          Deny from all
          Allow from 127.0.0.1
          </FilesMatch>
      </Directory>
      ErrorLog "logs/xtears.cn-error_log"
      CustomLog "|/usr/local/apache2.4/bin/rotatelogs -l logs/xtears.cn-access_%Y%m%d.log 86400" common
  </VirtualHost>
  ```

- `admin.php(.*)`是为了匹配地址后面带有参数的情况，测试如下：

  ```bash
  # curl -x118.24.153.130:80 'xtears.cn/admin.php?asd' -I
  HTTP/1.1 403 Forbidden
  Date: Thu, 31 May 2018 16:13:04 GMT
  Server: Apache/2.4.33 (Unix) PHP/5.6.32
  Content-Type: text/html; charset=iso-8859-1
  
  # curl -x127.0.0.1:80 'xtears.cn/admin.php?asd' -I
  HTTP/1.1 200 OK
  Date: Thu, 31 May 2018 16:13:22 GMT
  Server: Apache/2.4.33 (Unix) PHP/5.6.32
  X-Powered-By: PHP/5.6.32
  Content-Type: text/html; charset=UTF-8
  ```

## 其他访问控制

- 禁止访问指定目录/文件，如不允许访问.inc扩展名文件，保护php类库，虚拟主机配置增加如下内容:

  ```bash
  <Files ~ "\.inc$">
      Order allow,deny
      Deny from all
  </Files>
  ```

- 禁止访问指定目录：

  ```bash
  <Directory ~ "^/var/www/(.+/)*[0-9]{3}">
      Order allow,deny
      Deny from all
  </Directory>
  
  ```

- 通过文件匹配来禁止，如禁止访问图片：

  ```bash
  <FilesMatch \.(gif|jpe?g|png)$>
      Order allow,deny
      Deny from all
  </FilesMatch>
  
  ```

- 针对URL相对路径的访问控制：

  ```bash
  <Location /dir/>
      Order allow,deny
      Deny from all
  </Location>
  ```

- 禁止某些IP访问：

  ```bash
  <Directory "/var/www/web/">
  	Order allow,deny
  	Allow from all
  	Deny from 10.0.0.1 #阻止ip
  	Deny from 192.168.0.0/24 #阻止ip段
  </Directory>
  
  ```

- 允许某些IP访问，适合内部或合作公司访问:

  ```bash
  <Directory "/var/www/web/">
  	Order deny,allow
  	Deny from all
  	All from example.com #允许域名
  	All from 10.0.0.1	#允许一个IP
  	All from 10.0.0.1 10.0.0.2	#允许多个IP
  	Allow from 10.1.0.0/255.255.0.0	#允许一个IP段、掩码对
  	All from 10.0.1 192.168	#允许一个IP段，ip段后面不写
  	All from 192.168.0.0/24
  </Directory>
  
  ```

# Apache禁止trace

- 禁止Trace和Track这两种调试web服务器链接的http凡是，可以防止xss攻击，具体可以使用rewrite功能实现：

  ```bash
  RewriteEngine On
  RewriteCond %{REQUEST_METHOD}^TRACE
  RewriteRule .* - [F]
  
  ```

  - 或者再apache配置文件中配置`TraceEnable off`参数项。

# Apache配置https

- 首先下载openssl源码包并使用默认配置编译，默认将安装到`/usr/local/ssl`：

  ```bash
  tar -zxf openssl-0.9.8k.tar.gz
  cd openssl-0.9.8k
  ./config   
  make && make install
  ```

- 然后编译apache指定ssl支持，可以编译为静态模块或动态模块：

  - 静态：`--enable-ssl=static --with-ssl=/usr/local/ssl`
  - 动态：`--enable-ssl=shared --with-ssl=/usr/local/ssl`
  - 动态编译的情况会在module目录生成`mod_ssl.so`文件，并且需要在httpd.conf中加入`LoadModule ssl_module modules/mod_ssl.so`。

- 接着创建私钥：

  ```bash
  cd /usr/local/ssl/bin
  openssl genrsa -out server.key 2048
  //如果需要对server.key添加保护密码，使用 -des3 扩展命令。Windows环境下不支持加密格式私钥，Linux环境下使用加密格式私钥时，每次重启Apache都需要您输入该私钥密码（例：openssl genrsa -des3 -out server.key 2048）
  cp server.key /usr/local/apache/conf/ssl.key/
  
  ```

- 之后生成证书请求(CSR)文件：

  ```bash
  # openssl req -new -key server.key -out certreq.csr
  Country Name：                           //所在国家的ISO标准代号，中国为CN   
  State or Province Name：                 //单位所在地省/自治区/直辖市   
  Locality Name：                          //单位所在地的市/县/区   
  Organization Name：                      //单位/机构/企业合法的名称   
  Organizational Unit Name：               //部门名称   
  Common Name：                            //通用名，例如：www.itrus.com.cn。此项必须与访问提供SSL服务的服务器时所应用的域名完全匹配。   
  Email Address：                          //邮件地址，不必输入，直接回车跳过   
  "extra"attributes                        //以下信息不必输入，回车跳过直到命令执行完毕。 
  ```

- 将生成的certreq.csr文件提交到证书申请机构，等待CA证书签发，服务器证书密钥对必须配对使用，私钥丢失会导致证书不可用。

- 将签发的两张中级CA证书内容保存到`conf/ssl.crt/intermediatebundle.crt`文件中，并将服务器证书内容保存到`ssl.crt/server.crt`文件中。

- 在httpd.conf中增加以下内容：

  ```bash
  Listen 443
  NameVirtualHost *:443
  	
  	DocumentRoot "/data/web/www"
  	ServerName aaa.com:443
  	ErrorLog "logs/error.log"
  	CustomLog "logs/access.log" combined
  	
  		SSLEngine on
  		SSLCertificateFile /usr/local/apache/conf/ssl.crt/server.crt
  		SSLCertificateKeyFile /usr/local/apache/conf/ssl.key/server.key
  		SSLCertificateChainFile /usr/local/apache/conf/ssl.crt/intermediatebundle.crt
  ```

---

