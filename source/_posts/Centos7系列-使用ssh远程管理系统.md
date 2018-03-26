---
title: 'Centos7系列:使用ssh远程管理系统'
author: Evobot
abbrlink: cb3f93e8
date: 2018-03-21 22:59:10
categories: Centos7
tags: [Linux, Centos]
image: http://p5qynomrl.bkt.clouddn.com/1521646685090x27m0c8b.png?imageslim
photo:
---

虚拟机里安装的系统，无法使用鼠标，无法复制粘贴，使用起来不太方便，所以一般都会使用SSH客户端来管理Linux系统，这里使用PuTTY和Xshell来远程连接Centos7，并且使用密钥认证来保证连接的安全。

<!-- more -->

# 使用PuTTY和Xshell远程管理系统

## PuTTY的使用

- PuTTY是一个SSH/Telnet客户端，可以让我们方便的登陆管理远程机器，可以在这里[下载](https://the.earth.li/~sgtatham/putty/latest/w32/putty-0.70-installer.msi)。


- 打开软件，界面如下：

  ![PuTTY](http://p5qynomrl.bkt.clouddn.com/1521645606365crhs2ghy.png?imageslim)

- PuTTY的具体设置可以参考这篇博文：[Putty工具保存配置的小技巧](http://blog.csdn.net/tianlesoftware/article/details/5831605)

- 点击**Open**就可以看到登陆窗口：

  ![登陆窗口](http://p5qynomrl.bkt.clouddn.com/15216461185495xjxn33b.png?imageslim)

- 与在虚拟机中登陆相同，输入用户名root和密码，就可以登陆到Centos系统。

## Xshell的使用

- Xshell与PuTTY相比，功能更强大，界面也更美观。

- Xshell打开时会显示会话窗口，我们点击新建来创建一个新的连接：

  ![Xshell](http://p5qynomrl.bkt.clouddn.com/1521646685090x27m0c8b.png?imageslim)

- 新建会话如下，填写主机的名称，IP地址以及端口号，SSH协议默认为22端口：

  ![新建会话](http://p5qynomrl.bkt.clouddn.com/1521646893899i9spdd1t.png?imageslim)

- 其余我们可以设置用户身份验证，输入用户面密码或者使用密钥认证，还要外观配置等等，更加详细的Xshell配置，可以参考这篇博文：[Xshell学习](https://www.cnblogs.com/perseverancevictory/p/4910145.html)

# 密钥认证

- 密钥认证使用一对公钥和私钥来对连接主机的会话进行安全认证，在服务器上放置公钥，本地电脑放置私钥，这样在以后连接时，就只有与公钥配对的私钥主机能够连接服务器，以确保登陆的用户是可信任的。

## PuTTY使用密钥认证

- 安装PuTTY时，默认安装的有`PuTTYgen`软件，打开软件，点击**Generate**来生成密钥对,在生成时需要晃动鼠标加快生成速度：

  ![PuTTYgen](http://p5qynomrl.bkt.clouddn.com/1521648026211ycih6g5o.png?imageslim)

- 如图所示，红框1内就是生成的公钥，点击红框2的**Save private key**，将私钥保存到本地。

  ![生成密钥](http://p5qynomrl.bkt.clouddn.com/1521648266102nqn3osvx.png?imageslim********)

## 添加公钥到Centos

- 登陆到主机，创建隐藏目录`.ssh`,我们的公钥都需要写入这个目录下的`authorize_keys`文件内：

```bash
[root@localhost ~]# mkdir .ssh
[root@localhost ~]# chmod 700 .ssh/	# 更改目录权限
[root@localhost ~]# touch .ssh/authorized_keys
```

- 然后打开`authorized_keys`文件，将生成的公钥字符串粘贴到文件内保存退出；
- 之后还需要关闭`selinux`，使用命令`setenforce 0`临时关闭selinux；
- 接下来在PuTTY中配置会话的私钥，如下图，选中已经保存的私钥，然后在session内保存即可：

  ![加载私钥](http://p5qynomrl.bkt.clouddn.com/1521649098132szmq5m5r.png?imageslim)

- 点击Open，输入用户名root，这时就不用输入密码即可登陆到系统：
  
  ![密钥登陆](http://p5qynomrl.bkt.clouddn.com/1521649365052zfmeg2wi.png?imageslim)

## Xshell使用密钥认证

- Xshell生成密钥只需要在窗口菜单栏点击**工具**-**新建用户密钥生成向导**，点击两次下一步到设置密钥的名称以及密钥加密密码：

  ![Xshell生成密钥](http://p5qynomrl.bkt.clouddn.com/1521649669179f475f92v.png?imageslim)

- 点击下一步会显示生成的公钥字符串，与之前添加公钥的方式相同，到Centos中添加公钥。

- 然后在Xshell的窗口中打开会话属性，在用户身份验证中配置私钥即可：

  ![Xshell密钥登陆](http://p5qynomrl.bkt.clouddn.com/15216501748407pz0kf1w.png?imageslim)

- 之后我们点击连接就不再需要输入用户名密码即可登陆。

---

