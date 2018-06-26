---
title: 安装jdk和Tomcat
author: Evobot
categories: Tomcat
tags:
  - Tomcat
  - jdk
  - Centos7
image: 'https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/tomcat.png'
abbrlink: af3c490c
date: 2018-06-26 21:29:21
---



本文主要对Tomcat进行介绍，并演示了如何安装JDK环境和Tomcat环境。

<!--more-->

---

# Tomcat介绍

- Tomcat是Apache软件基金会的Jakarta项目中的一个核心项目，由Apache、Sun等公司共同开发而成；
- LAMP和LNMP都是针对PHP开发的网站的运行环境，而Tomcat实际上是一种中间件，是针对java程序写的网站，需要Tomcat+jdk来运行，真正起解析Java脚本的是jdk；
- jdk(Java development kit)是整个Java的核心，包含了Java运行环境和相关工具、基础库；
- 主流的jdk为sun公司发布的jdk，除此之外，IBM等公司也有发布的JDK，Centos上也有可以用yum安装的开源的OpenJDK。

# JDK安装

- jdk版本目前有1.6、1.7、1.8，也简称为6、7、8版本；

- 我们使用1.8版本，下载JDK1.8，到[jdk官网](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)下载Linux x64版本的tar.gz安装包，然后上传到Centos上并解压；

- 将解压出来的目录移动到`/usr/local`下并重命名为jdk1.8：

  ```bash
  $ mv jdk1.8.0_171/ /usr/local/jdk1.8
  ```

- 在`/etc/profile`文件末尾添加JDK的环境变量：

  ```bash
  JAVA_HOME=/usr/local/jdk1.8
  JAVA_BIN=/usr/local/jdk1.8/bin
  JRE_HOME=/usr/local/jdk1.8/jre
  PATH=$PATH:/usr/local/jdk1.8/bin:/usr/local/jdk1.8/jre/bin
  CLASSPATH=/usr/local/jdk1.8/jre/lib:/usr/local/jdk1.8/lib:/usr/local/jdk1.8/jre/lib/charsets.jar
  ```

- 重新加载profile文件，执行`java -version`确认jdk环境是否安装成功：

  ```bash
  $ source /etc/profile
  $ java -version
  java version "1.8.0_171"
  Java(TM) SE Runtime Environment (build 1.8.0_171-b11)
  Java HotSpot(TM) 64-Bit Server VM (build 25.171-b11, mixed mode)

  ```

# Tomcat安装

- 下载Tomcat二进制安装包：

  ```bash
  $ wget http://mirrors.hust.edu.cn/apache/tomcat/tomcat-8/v8.5.31/bin/apache-tomcat-8.5.31.tar.gz
  ```

- 解压Tomcat压缩包并移动到`/usr/local`下重命名为**tomcat**：

  ```bash
  $ tar zxvf apache-tomcat-8.5.31.tar.gz -C /usr/local/

  $ mv /usr/local/{apache-tomcat-8.5.31,tomcat}
  ```

- 然后就可以启动Tomcat，并查看系统进程：

  ```bash
  $ /usr/local/tomcat/bin/startup.sh 
  Using CATALINA_BASE:   /usr/local/tomcat
  Using CATALINA_HOME:   /usr/local/tomcat
  Using CATALINA_TMPDIR: /usr/local/tomcat/temp
  Using JRE_HOME:        /usr/local/jdk1.8
  Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar
  Tomcat started.

  $ ps aux |grep java
  root       2229  0.8 16.8 2170880 81632 pts/0   Sl   22:31   0:05 /usr/local/jdk1.8/bin/java -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -Djava.protocol.handler.pkgs=org.apache.catalina.webresources -Dorg.apache.catalina.security.SecurityListener.UMASK=0027 -Dignore.endorsed.dirs= -classpath /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar -Dcatalina.base=/usr/local/tomcat -Dcatalina.home=/usr/local/tomcat -Djava.io.tmpdir=/usr/local/tomcat/temp org.apache.catalina.startup.Bootstrap start

  ```

- Tomcat不支持restart，所以在更改了配置文件后，需要先停止服务再重新启动，停止服务的命令如下：

  ```bash
  $ /usr/local/tomcat/bin/shutdown.sh 
  Using CATALINA_BASE:   /usr/local/tomcat
  Using CATALINA_HOME:   /usr/local/tomcat
  Using CATALINA_TMPDIR: /usr/local/tomcat/temp
  Using JRE_HOME:        /usr/local/jdk1.8
  Using CLASSPATH:       /usr/local/tomcat/bin/bootstrap.jar:/usr/local/tomcat/bin/tomcat-juli.jar

  ```

- 重新启动服务后，查看Tomcat启动的Java进程监听的端口如下：

  ```bash
  $ netstat -tlnp | grep java
  tcp6       0      0 :::8080                 :::*                    LISTEN      2321/java           
  tcp6       0      0 127.0.0.1:8005          :::*                    LISTEN      2321/java           
  tcp6       0      0 :::8009                 :::*                    LISTEN      2321/java   
  ```

  - 8080端口是提供web服务的端口，在浏览器中测试访问会显示如下页面：

  ![tomcat](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/tomcat.png)

  - 8005为管理端口，8009则是第三方服务调用的端口，比如httpd和Tomcat结合使用时会用到，但是很少使用。

---