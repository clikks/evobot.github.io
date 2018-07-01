---
title: Tomcat相关配置
author: Evobot
date: 2018-06-27 21:41:57
categories: Tomcat
tags:
  - Tomcat
  - Centos
  - zrlog
image:
---



本文通过部署zrlog博客系统，来介绍如何Tomcat的虚拟主机配置和日志相关的配置。

<!--more-->

---

# 配置Tomcat监听80端口

- Tomcat默认监听的时8080端口，使得访问起来不够方便，而修改Tomcat的监听端口需要更改配置文件`/usr/local/tomcat/conf/server.xml`，然后搜索8080，将其改成80并重启服务，配置文件更改如下：

  ```xml
  <Connector port="80" protocol="HTTP/1.1"
                 connectionTimeout="20000"
                 redirectPort="8443" />
      <!-- A "Connector" using the shared thread pool-->
  ```

  重启服务后查看是否监听到80端口：

  ```bash
  $ netstat -tlnp | grep java
  tcp6       0      0 :::80                   :::*                    LISTEN      2054/java           
  tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      2054/java           
  tcp6       0      0 :::8009                 :::*                    LISTEN      2054/java     
  ```

# Tomcat虚拟主机

## 增加虚拟主机配置

- Tomcat同样支持虚拟主机，编辑配置文件`server.xml`，查找`host`配置项：

  ```xml
  <Host name="localhost"  appBase="webapps"
              unpackWARs="true" autoDeploy="true">
    <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
           prefix="localhost_access_log" suffix=".txt"
           pattern="%h %l %u %t &quot;%r&quot; %s %b" />
  </Host>

  ```

- 与nginx和Apache不同，Tomcat的虚拟主机不是提供root网站根目录，而是提供WAR格式的包，包里包含站点的文件、目录、JSP文件等等，并且需要将包放在上面host配置中`appBase`指定的目录中，而`unpackWARs`配置为true，则Tomcat会自动解压WAR包；

- 除了提供WAR包，也可以单独指定一个目录，目录内放站点的根目录；

- 上面的`valve`配置项则是日志格式的配置项；

- 增加一个虚拟主机需要在配置中增加一项host配置项，如下增加虚拟主机配置：

  ```xml
  <Host name="www.123.cn" appBase=""
        unpackWARS="true" autoDeploy="true"
        xmlValidation="false" xmlNamespaceAware="false">
        <Context path="" docBase="/data/wwwroot/123.cn/" debug="0"
  reloadable="true" crossContext="true"/>
  </Host>
  ```

  - 这里没有指定appBase，而是在Context配置项中指定了`docBase`，这就是定义的目录，用来存放网站根目录，也可以将WAR手动解压后放入docBase目录；
  - 如果不定义docBase，那么默认是在appBase/ROOT目录下，定义了docBase后，就以docBase为主，并且appBase和docBase可以一样；

## 部署站点

- 部属站点我们使用zrlog博客系统，是由java构建的，首先下载[zrlog](http://dl.zrlog.com/release/zrlog-2.0.0-4602099-release.war?attname=ROOT.war&ref=index)；

- 将zrlog的WAR包拷贝到`/usr/local/tomcat/webapps/`目录下，然后等待一会查看webapps目录，会看到zrlog的WAR包被自动解压出来，这是`unpackWARS="true"`配置的作用：

  ```bash
  $ cp zrlog.war /usr/local/tomcat/webapps/

  $ ls /usr/local/tomcat/webapps/
  docs  examples  host-manager  manager  ROOT  zrlog  zrlog.war

  ```

- 自动解压完成后，不要删除原始的WAR包，否则自动解压出来的目录也会被删除，而将解压的目录更名，则tomcat会再次解压出与WAR同名的目录；

- 然后使用浏览器访问zrlog的安装向导：

  ![zrlog](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zrlog.png)

- 安装向导首先要求配置数据库，我们登录MySQL，为zrlog创建一个名为`zrlog`的数据库，并且创建同名的用户，指定权限只能访问zrlog库：

  ```sql
  mysql> create database zrlog;
  Query OK, 1 row affected (0.01 sec)

  mysql> grant all on zrlog.* to 'zrlog'@127.0.0.1 identified by '123456';
  Query OK, 0 rows affected, 1 warning (0.02 sec)

  ```

- 在安装向导中填入数据库信息之后点击下一步，设置网站信息后安装完成：

  ![zrlog-site](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zrlog-web.png)

## zrlog虚拟主机配置完善

- 上一步安装zrlog中，我们访问安装向导时，访问的URL是192.168.199.128/zrlog，而不是像Apache和nginx一般直接访问虚拟主机的Server_name；

- 这是因为我们的zrlog的网站目录放在了webapps目录下，而我们配置文件中zrlog的虚拟主机配置，对应的网站根目录为docBase目录，所以我们将**zrlog的根目录下的所有文件**移动到docBase指定的目录下，即`/data/wwwroot/123.cn/`目录：

  ```bash
  $ mkdir -p /data/wwwroot/123.cn  

  $ mv /usr/local/tomcat/webapps/zrlog/* /data/wwwroot/123.cn/

  ```

- 由于站点是在虚拟机中，而域名配置的是`www.123.cn`，所以需要配置hosts，将域名指向虚拟机；

- 重启Tomcat服务，然后直接在浏览器中访问`www.123.cn`域名，查看是否能够访问到zrlog：

  ![zrlog-domian](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zrlog-domain.png)

- 在Tomcat的server.xml中，appBase的目录webapps是以相对路径配置的，相对路径是相对Tomcat安装目录的位置；而webapps下的ROOT目录就是Tomcat安装后的默认访问页面，ROOT下的文件可以用`localhost/index.jsp`的形式进行访问；

- 如果在配置文件中自定义了appBase的路径，那么也需要在自定义的路径下创建ROOT目录，并且将图片、静态文件等放到ROOT下才能正常访问；

# Tomcat日志

- Tomcat的日志默认在`/usr/local/tomcat/logs`目录下：

  ```bash
  $ ls /usr/local/tomcat/logs/
  catalina.2018-06-26.log      localhost.2018-06-27.log
  catalina.2018-06-27.log      localhost.2018-06-28.log
  catalina.2018-06-28.log      localhost_access_log.2018-06-26.txt
  catalina.out                 localhost_access_log.2018-06-27.txt
  host-manager.2018-06-26.log  localhost_access_log.2018-06-28.txt
  host-manager.2018-06-27.log  manager.2018-06-26.log
  host-manager.2018-06-28.log  manager.2018-06-27.log
  localhost.2018-06-26.log     manager.2018-06-28.log

  ```

  ​

- 其中`catalina`开头的是综合日志，记录了Tomcat服务相关的信息和错误信息:

  ```bash
  $ tail catalina.out 
  28-Jun-2018 00:18:30.261 信息 [www.123.cn-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/tomcat/webapps] has finished in [126] ms
  28-Jun-2018 00:18:30.261 信息 [www.123.cn-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deploying web application directory [/usr/local/tomcat/work]
  28-Jun-2018 00:18:30.317 信息 [www.123.cn-startStop-1] org.apache.catalina.startup.HostConfig.deployDirectory Deployment of web application directory [/usr/local/tomcat/work] has finished in [56] ms
  28-Jun-2018 00:18:30.427 信息 [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-80"]
  28-Jun-2018 00:18:30.757 信息 [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["ajp-nio-8009"]
  28-Jun-2018 00:18:30.762 信息 [main] org.apache.catalina.startup.Catalina.start Server startup in 16836 ms
  六月 28, 2018 12:18:30 上午 com.zrlog.plugincore.server.impl.NioServer create
  信息: plugin listening on port -> 45941
  六月 28, 2018 12:18:30 上午 com.zrlog.plugincore.server.impl.NioServer create
  信息: load jar files

  ```

  - 主要需要关注错误、严重级别的日志，而catalina.2018-xx-xx-xx.log与catalina.out内容相同，只是前者会每天生成一个新日志；

- host-manager.xxxx-xx-xx.log和manager日志是管理相关日志，host-manager是虚拟主机管理日志，一般较少使用；

- localhost和localhost_access是虚拟主机相关日志，有access命名的是访问日志，反之则是虚拟主机错误日志：

  ```bash
  $ tail -n3 localhost.2018-06-28.log 
  28-Jun-2018 00:18:23.303 信息 [localhost-startStop-1] org.apache.catalina.core.ApplicationContext.log ContextListener: contextInitialized()
  28-Jun-2018 00:18:23.303 信息 [localhost-startStop-1] org.apache.catalina.core.ApplicationContext.log SessionListener: contextInitialized()
  28-Jun-2018 00:18:23.305 信息 [localhost-startStop-1] org.apache.catalina.core.ApplicationContext.log ContextListener: attributeAdded('StockTicker', 'async.Stockticker@60be3b74')

  $ tail -n3 localhost_access_log.2018-06-28.txt 
  192.168.199.199 - - [28/Jun/2018:00:19:27 +0800] "GET /asf-logo-wide.svg HTTP/1.1" 304 -
  192.168.199.199 - - [28/Jun/2018:00:19:27 +0800] "GET /bg-upper.png HTTP/1.1" 304 -
  192.168.199.199 - - [28/Jun/2018:00:19:50 +0800] "GET / HTTP/1.1" 200 11250

  ```

- 访问日志默认不会生成，需要在server.xml中配置，例如配置zrlog虚拟主机的访问日志，在`<Host></Host>`中添加如下配置：

  ```xml
  <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
     prefix="123.cn_access" suffix=".log"
     pattern="%h %l %u %t &quot;%r&quot; %s %b" />
  ```

  - `prefix`定义日志文件名的前缀，`suffix`定义日志文件后缀名，`pattern`定义日志格式，`directory`定义日志存储目录。
  - 新增加的虚拟主机默认不会生成类似默认虚拟主机的localhost.data.log日志，错误日志会统一记录到catalina.out中，所以Tomcat出现问题时，应该第一时间查看catalina.out日志。


---

