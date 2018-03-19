title: Centos7单用户模式修改root密码
author: Evobot
tags:
  - Centos
  - Linux
categories:
  - Linux
date: 2017-08-27 21:17:00
image: "http://p5qynomrl.bkt.clouddn.com/1521312295772z2f2ptcr.png?imageslim"
---
对于忘记root密码的情况，我们可以进入Centos的单用户模式，来修改我们的root密码，下面为如何进入单用户模式并修改root密码的详细步骤。
<!--more-->
# Centos7单用户模式
## 配置单用户模式
1. 在Centos7启动界面下，对第一个启动项按**e**键进入配置界面。
![paste image](http://p5qynomrl.bkt.clouddn.com/1521312295772z2f2ptcr.png?imageslim)
2. 将光标定位到**linux16**行，再将光标移动到**ro**位置。
![paste image](http://p5qynomrl.bkt.clouddn.com/15213123237461a612knk.png?imageslim)
3. 将**ro**只读修改为**rw**读写模式，并且添加**init=/sysroot/bin/sh**在**rw**后面。
![paste image](http://p5qynomrl.bkt.clouddn.com/1521312341229joes7584.png?imageslim)
4. **sysroot**就是我们原先的系统root路径，完成之后，按**Ctrl+x**键，保存退出。
## 进入单用户模式
- 进入到系统后，需要切换到我们原有系统的环境下再继续操作，所以首先执行`chroot /sysroot/`命令切换到我们原系统**root**环境。
```bash
:/# chroot /sysroot/
```
# 修改root密码
## 修改系统语言编码
- 为了防止出现乱码的情况，首先修改系统语言环境为英文。
```bash
:/# LANG=en
```
## 修改密码
- 这时候我们执行`passwd root`来修改root的密码。
```bash
:/# passwd root
Changing password for user root.
New password:
Retype new password:
```
## 更新**autorelabel**文件
- 修改完密码后，还需要创建**autorelabel**文件，这样重启才能进入系统。
```bash
:/# touch /.autorelabel
```
> 执行完所有操作后重启计算机即可使用新的密码登陆root账户。
---
