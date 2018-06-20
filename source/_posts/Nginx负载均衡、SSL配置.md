---
title: Nginx负载均衡、SSL配置
author: Evobot
categories: LNMP
tags:
  - Linux
  - Centos
  - Nginx
abbrlink: ebec8c28
date: 2018-06-12 01:11:45
image:
---



本文主要介绍Nginx负载均衡配置，以及SSL原理、如何生成ssl密钥对，最后介绍Nginx如何配置ssl。

<!--more-->

---

# Nginx负载均衡

## 负载均衡配置

- 实际上负载均衡就是对代理的运用，代理一台web服务器成为代理，而代理多台服务器，则成为负载均衡，负载均衡能够在多台服务器中有一台down掉无法提供服务时，将请求发给其他的服务器,nginx配置负载均衡需要使用`upstream`模块；

- 安装`bind-utils`软件包，使用其提供的`dig`命令，如`dig qq.com`，命令会返回域名的ip地址：

  ```bash
  [root@evobot vhost]# dig qq.com
  
  ; <<>> DiG 9.9.4-RedHat-9.9.4-61.el7 <<>> qq.com
  ;; global options: +cmd
  ;; Got answer:
  ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 2474
  ;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1
  
  ;; OPT PSEUDOSECTION:
  ; EDNS: version: 0, flags:; udp: 4096
  ;; QUESTION SECTION:
  ;qq.com.                                IN      A
  
  ;; ANSWER SECTION:
  qq.com.                 135     IN      A       111.161.64.48
  qq.com.                 135     IN      A       111.161.64.40
  
  ;; Query time: 59 msec
  ;; SERVER: 100.88.222.14#53(100.88.222.14)
  ;; WHEN: 二 6月 12 01:39:21 CST 2018
  ;; MSG SIZE  rcvd: 67
  ```

- 创建一个新的虚拟主机配置，如load.conf，这里对qq.com进行负载均衡配置，写入下面的配置：

  ```bash
  upstream qq_com
  {
      ip_hash;
      server 111.161.64.48;
      server 111.161.64.40;
  }
  server
  {
      listen 80;
      server_name www.qq.com;
      location /
      {
          proxy_pass  http://qq_com;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP  $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }
  ```

  > 这里upstream后面的qq_com为自定义的名字，而ip_hash是为了在正常情况下让用户始终访问到固定的一个服务器，而这里的proxy_pass不再写代理的ip，而是写upstream的自定义名字，这里定义的是qq_com，其余配置与代理配置相同。

- 重新加载配置后，使用`curl -x127.0.0.1:80 www.qq.com`访问本机就可以访问到qq.com的主页；

- 还需要注意的是，nginx不支持代理https，即代理443端口，如果要实现https，只能让nginx监听443端口，配置为https，然后负载均衡的后端服务器则监听80端口，这样的方式实现https。

## 根据请求uri进行代理

- 加入一个需求为使用一台Nginx代理4台Apache服务器，并且是根据不同的uri，代理到不同的服务器上，那么需要的配置如下：

  ```bash
  upstream aa.com {
    server 192.168.0.121;
    server 192.168.0.122;
  }

  upstream bb.com {
    server 192.168.0.123;
    server 192.168.0.124;
  }

  server {
    listen    80;
    server_name   www.abc.com;
    location ~ aa.php {
      proxy_pass http://aa.com/;
      proxy_set_header Host   $host;
      proxy_set_header X-Real-IP  $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location ~ bb.php {
      proxy_pass http://bb.com/;
      proxy_set_header Host   $host;
      proxy_set_header X-Real-IP  $remote_addr;
      proxy_set_header X-Forwarded-For    $proxy_add_x_forwarded_for;
    }
  }
  ```

## 根据访问目录代理

- 如用户请求目录`/aaa/`时，需要将请求发送到a机器，而请求目录为`/bbb/`时，则把请求发送到b机器，其余请求同样发送到b机器，那么nginx的配置如下：

  ```bash
  upstream aaa.com {
    server 192.168.111.6;
  }
  
  upstream bbb.com {
    server 192.168.111.20;
  }
  
  server {
    listen 80;
    server_name li.com;
    
    location /aaa/ {
      proxy_pass http://aaa.com/aaa/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP    $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
    
    location /bbb/ {
      proxy_pass http://bbb.com/bbb/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP    $remote_addr;
      proxy_set_header X-Forwarede-For  $proxy_add_x_forwarded_for;
    }
    
    location / {
      proxy_pass http://bbb.com/;
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP    $remote_addr;
      proxy_set_header X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
  }
  ```
  > 这里的`location /bbb/`可以省略，因为后面的`location /`已经包含了/bbb/。

# SSL配置

## SSL原理

- 我们在访问网站的时候，如果使用`https://`开头，则会使用加密的通信来访问网站，提高了安全性，SSL的流程图如下：

  ![SSL流程](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/SSL1.png)

1. 浏览器发送一个https的请求给服务器，服务器有自己的一套数字证书，证书可以自己制作，也可以向组织申请，但自己颁发的证书，浏览器会弹出提示页面表示证书不受信任；
2. 服务器收到请求后，会将公钥传输给客户端；
3. 客户端即浏览器在收到公钥后，会验证其合法性，无效会警告提示，有效则会生成一串随机数，并使用收到的公钥进行加密；
4. 客户端将加密后的随机字符串传输给服务器；
5. 服务器收到加密随机字符后，首先使用私钥解密(公钥加密，私钥解密)，获取到这一串随机数后，再用这串随机字符串加密要传输的数据(对称加密，即将数据和私钥也就是这串随机字符串通过算法混合，这样除非知道私钥，否则无法获取数据内容)；
6. 服务器将加密后的数据传输给客户端后，客户端使用之前的随机字符串进行解密，完成数据的传输。

## 生成SSL密钥对

- 自己生成SSL密钥对，所需要的操作如下：

  ```bash
  cd /usr/local/nginx/conf/
  #使用openssl生成私钥，如果没用命令，则需要安装openssl软件包
  #这里需要输入私钥的密码
  [root@evobot conf]# openssl genrsa -des3 -out tmp.key
  Generating RSA private key, 2048 bit long modulus
  .......................+++
  ..............................+++
  e is 65537 (0x10001)
  Enter pass phrase for tmp.key:
  Verifying - Enter pass phrase for tmp.key:
  ```

- 然后为私钥转换格式，将私钥的密码去除，并且删除之前生成的私钥：

  ```bash
  [root@evobot conf]# openssl rsa -in tmp.key -out evobot.key
  Enter pass phrase for tmp.key:
  writing RSA key
  
  [root@evobot conf]# rm -rf tmp.key
  ```

- 接着生成证书请求文件，并用这个文件与私钥一起生成公钥文件,在生成的过程中，需要多次输入一些国家、地址、email等信息：

  ```bash
  [root@evobot conf]# openssl req -new -key evobot.key -out evobot.csr
  You are about to be asked to enter information that will be incorporated
  into your certificate request.
  What you are about to enter is what is called a Distinguished Name or a DN.
  There are quite a few fields but you can leave some blank
  For some fields there will be a default value,
  If you enter '.', the field will be left blank.
  -----
  Country Name (2 letter code) [XX]:86
  State or Province Name (full name) []:Sichuan
  Locality Name (eg, city) [Default City]:Chengdu
  Organization Name (eg, company) [Default Company Ltd]:evobot.cn
  Organizational Unit Name (eg, section) []:evobot
  Common Name (eg, your name or your server's hostname) []:evobot
  Email Address []:admin@evobot.cn
  
  Please enter the following 'extra' attributes
  to be sent with your certificate request
  A challenge password []:evobot
  An optional company name []:evobot
  ```

- 最后使用上面的文件和私钥来生成公钥文件：

  ```bash
  [root@evobot conf]# openssl x509 -req -days 365 -in evobot.csr -signkey evobot.key -out evobot.crt
  Signature ok
  subject=/C=86/ST=Sichuan/L=Chengdu/O=evobot.cn/OU=evobot/CN=evobot/emailAddress=admin@evobot.cn
  Getting Private key
  
  ```

- 最终我们得到的有以下三个文件：

  ```bash
  [root@evobot conf]# ls evobot.*
  evobot.crt  evobot.csr  evobot.key
  ```

## Nginx配置SSL

- 创建一个新的虚拟主机配置文件，如ssl.conf，写入以下配置：

  ```bash
  server
  {
      listen 443;
      server_name auth.evobot.cn;
      index index.html index.php;
      root /data/wwwroot/auth.evobot.cn;
      ssl on;
      ssl_certificate evobot.crt;
      ssl_certificate_key evobot.key;
      ssl_protocols TLSv1 TLSv1.1 TLSV1.2;
  }
  ```

- 这里监听443端口，并且将ssl置为on，然后指定ssl的公钥和key，最后指定ssl的协议，一般将三种协议均指定。

- 重新加载配置报错：

  ```bash
  [root@evobot vhost]# /usr/local/nginx/sbin/nginx -t
  nginx: [emerg] unknown directive "ssl" in /usr/local/nginx/conf/vhost/ssl.conf:7
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test failed
  ```

  > 这是因为我们在编译nginx时没用编译ssl模块，所以需要重新对nginx进行编译安装。

- 重新编译nginx，查看nginx的编译参数：

  ```bash
  [root@evobot nginx-1.12.2]# ./configure --help | grep -i ssl
    --with-http_ssl_module             enable ngx_http_ssl_module
    --with-mail_ssl_module             enable ngx_mail_ssl_module
    --with-stream_ssl_module           enable ngx_stream_ssl_module
    --with-stream_ssl_preread_module   enable ngx_stream_ssl_preread_module
    --with-openssl=DIR                 set path to OpenSSL library sources
    --with-openssl-opt=OPTIONS         set additional build options for OpenSSL
  ```

  > 查询看到ssl需要使用编译参数`--with-http_ssl_module`。

  ```bash
  #重新编译
  ./configure --prefix=/usr/local/nginx --with-http_ssl_module
  
  make && make install
  
  #查看编译参数
  [root@evobot nginx-1.12.2]# /usr/local/nginx/sbin/nginx -V
  nginx version: nginx/1.12.2
  built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC)
  built with OpenSSL 1.0.2k-fips  26 Jan 2017
  TLS SNI support enabled
  configure arguments: --prefix=/usr/local/nginx --with-http_ssl_module
  ```

- 重新加载配置，不再报错，然后重启nginx服务：

  ```bash
  [root@evobot nginx]# /usr/local/nginx/sbin/nginx -t
  nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
  
  [root@evobot nginx]# service nginx restart
  Restarting nginx (via systemctl):                          [  确定  ]
  
  [root@evobot nginx]# netstat -tlnp | grep 443
  tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN      20768/nginx: master
  ```

- 然后访问站点，如果使用浏览器访问，需要添加iptables规则，将443端口放通：

  ```bash
  iptables -I INPUT -p tcp --dport 443 -j ACCEPT
  ```

  - 访问时会有警告提示，这是因为我们的证书是自己生成的，并不受浏览器信任：

  ![SSL访问](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/SSL2.png)

  - 使用curl访问，不能使用-x选项指定443端口，需要在hosts写入`127.0.0.1 auth.evobot.cn`，然后访问，仍然会有告警提示：

  ```bash
  [root@evobot nginx]# curl https://auth.evobot.cn
  curl: (60) Peer's certificate issuer has been marked as not trusted by the user.
  More details here: http://curl.haxx.se/docs/sslcerts.html
  
  curl performs SSL certificate verification by default, using a "bundle"
   of Certificate Authority (CA) public keys (CA certs). If the default
   bundle file isn't adequate, you can specify an alternate file
   using the --cacert option.
  If this HTTPS server uses a certificate signed by a CA represented in
   the bundle, the certificate verification probably failed due to a
   problem with the certificate (it might be expired, or the name might
   not match the domain name in the URL).
  If you'd like to turn off curl's verification of the certificate, use
   the -k (or --insecure) option.
  ```
---

  

  