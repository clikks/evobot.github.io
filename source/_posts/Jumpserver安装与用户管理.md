---
title: Jumpserver安装与用户管理
author: Evobot
categories: 堡垒机
tags:
  - Jumpserver
abbrlink: 6969a3ac
date: 2018-09-15 20:41:50
image:
---

1. jumpserver介绍
2. 安装jumpserver
3. 登录jumpserver
4. 创建管理用户

<!--more-->

---

# Jumpserver介绍

- Jumpserver是一款使用Python，Django开发的开元跳板机系统，助力互联网高效进行用户、资产、权限、审计管理；
- Jumpserver能够进行Auth统一认证、CMDB资产管理、统一授权、日志审计、自动化运维（ansible）；
- 目前Jumpserver使用的是Python3.6，django1.11；
- Jumpserver的官网地址为[http://www.jumpserver.org/](http://www.jumpserver.org/)。

# Jumpserver安装

- Jumpserver官方的[安装文档](http://docs.jumpserver.org/zh/docs/index.html)，提供了docker安装方式、傻瓜式安装以及生产环境正常安装三种方式，目前Jumpserver官方最新版本为1.4.1,这里我们安装的是0.3.2版本；

- 首先安装git软件包，然后克隆Jumpserver的软件仓库：

  ```bash
  git clone https://github.com/jumpserver/jumpserver.git
  ```

- 进入jumpserver目录，然后切换到0.3.2的tag版本，使用`git checkout -b [branch_name] [tag_name]`命令创建一个标签的分支，因为直接切换到标签，是不能修改标签内的代码的：

  ```bash
  git checkout -b v0.3.2 0.3.2
  ```

- 如果服务器内已经安装了MySQL，那么需要讲MySQL的编码改为utf8，否则安装Jumpserver时由于存在中文，会导致安装报错，编辑**my.cnf**文件，添加配置如下：

  ```bash
  [mysqld]
  character-set-server=utf8
  
  [client]
  default-character-set=utf8:
  ```

- 先进入MySQL，为Jumpserver创建数据库以及用户和密码：

  ```sql
  mysql> create database jumpserver;
  mysql> grant all on jumpserver.* to 'jumpserver'@'127.0.0.1' identified by 'jump123jump';
  ```

- 另外还要准备一个smtp邮箱，如163邮箱，然后关闭防火墙和selinux；

- 接着进入install目录，编辑`install.py`,将下面的代码进行修改：

  ```python
  @property
      def _is_redhat(self):
          # 将"centos"改为"centos linux"
          if self.dist == "centos" or self.dist == "redhat" or self.dist == "fedora" or self.dist == "amazon linux ami":
              return True
  
  ```

- 执行`python install.py`脚本，该脚本会自动安装Jumpserver，安装过程中会要求输入数据库账户密码、smtp服务器地址等配置信息，最后会提示配置管理员用户名和密码；

- 安装完成后，之前安装时输入的配置，其实在install的上层目录中的**jumpserver.conf**中也可以进行修改。

# 登录Jumpserver

- 在安装完成后可以直接访问`ip:8000`地址，进入到Jumpserver的web登录页面：

  ![Jumpserver-login](https://s1.ax1x.com/2018/09/16/iZC0TH.png)

- 输出之前安装时配置的管理员用户名密码登录Jumpserver，在主界面的左上角头像下的超级管理员字体上点击，选择修改信息可以对管理员密码进行修改：

  ![Jumpserver-main](https://s1.ax1x.com/2018/09/16/iZCf0g.png)

- 主要功能介绍：

  - 用户管理：用来定义允许登录Jumpserver平台的用户；
  - 资产管理：即CMDB，用来对主机、交换机等资产进行管理，如开发资产组、测试资产组等等，也可以对机房信息进行管理；
  - 授权管理：其中的系统用户菜单，就是记录跳板机登录客户机使用的登录用户相关的用户名、秘钥等信息的管理菜单；授权规则菜单，是用来配置指定用户或用户组具备连接指定机器权限规则的，例如开发组用户只能连接开发组的机器；
  - 在Jumpserver中有三种用户，一种是登录Jumpserver的用户，一种是登录客户机的用户，还有一种则是在设置菜单中的默认管理用户，  默认管理用户是用来在客户机上进行自动化或者执行一些操作时使用的用户，其在客户机上具有root权限或`NOPASSWD: ALL`的sudo权限，该用户只在Jumpserver到客户机上执行一些高权限操作时使用；
  - 日志审计：可以查看用户登录历史、执行命令历史、以及文件上传或下载历史；
  - 上传下载：可以上传文件到客户机或从客户机下载文件。

# 创建管理用户

- 管理用户需要在设置菜单中进行配置：

![iZPeNd.png](https://s1.ax1x.com/2018/09/16/iZPeNd.png)

- 需要注意的是这里的秘钥是用户的私钥，因为Jumpserver需要使用这个用户登陆客户机执行操作；

- 在web上填入自定义的用户名、密码（可以不填）、端口、以及私钥，私钥的生成建议直接在Jumpserver服务器上进行生成，进入家目录下的`.ssh`目录使用命令`ssh-keygen -f jump`，其中`-f`选项指定密钥对的名字：

  ```bash
  ssh-keygen -f jump
  [root@vm1 .ssh]# ls
  id_rsa  id_rsa.pub  jump  jump.pub  known_hosts
  
  ```

- 将私钥的内容粘贴到浏览器中的密钥输入框内，然后确认保存。

- 登陆客户机，创建一个与管理用户同名的用户，然后切换到该用户，并在其家目录下创建`.ssh`目录，将之前在Jumpserver上创建的公钥内容写入`authorized_keys`文件中：

  ```bash
  useradd jump
  su - jump
  mkdir .ssh
  vi .ssh/authorized_keys
  chmod 700 .ssh
  chmod 400 .ssh/authorized_keys
  ```

- 然后在Jumpserver服务器上尝试使用秘钥登陆客户机，如果没有提示输入密码即可登陆成功，则管理用户配置成功，在新添加客户机时，都需要在客户机上创建这个管理用户：

  ```bash
  ssh -i jump jump@192.168.49.129
  ```

---

