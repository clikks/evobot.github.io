---
title: windows10 & Arch 双系统安装指南【二】
tags:
  - linux
  - archlinux
  - cinnamon
  - windows10
categories: Linux
abbrlink: fc06cd7
date: 2016-02-14 20:53:46
---
![](http://images.evobot.cn/arch%20cinnamon%E6%B7%B1%E5%BA%A6%E6%88%AA%E5%9B%BE20160214223028.png)

在上一篇我们已经完成了基础系统的安装，目前开机archlinux只具有基础的命令行模式，本篇我们来完成一些基础配置和Cinnamon图形界面的安装。
<!-- more -->

---
# 基础设置
## 添加用户
　　新安装的系统只有一个root用户，日常使用root用户登陆系统会带来很大的安全隐患，所以，我们要添加一个用户，使用这个用户日常登陆系统：
```bash
#添加arch用户并添加到wheel用户组
useradd -m -G wheel -s /bin/bash arch
#为arch用户设置密码
passwd arch
#为wheel用户组配置sudo权限
visudo
#找到下面这一行，并去掉前面的#号
%wheel ALL=(ALL) ALL
```
```bash
#退出登陆，使用arch用户重新登陆系统
exit
```
## 添加Arch Linux中文社区仓库
Arch Linux 中文社区仓库 是由 Arch Linux 中文社区驱动的非官方用户仓库。包含中文用户常用软件、工具、字体/美化包等。
```bash
$ sudo vi /etc/pacman.conf
#在末尾添加以下内容
[archlinuxcn]
SigLevel = Optional TrustAll
Server = https://mirrors.ustc.edu.cn/archlinuxcn/$arch
```
```bash
#刷新pacman数据库
$ sudo pacman -Syy
#安装archlinuxcn-keyring 包以导入 GPG key
$ sudo pacman -S archlinuxcn-keyring
```
# 基础软件安装
　　现在我们安装一些基本的软件：
```bash
$ sudo pacman -S bash-completion yaourt 
```
# Cinnamon图形界面安装
## xorg、显卡驱动安装
首先安装xorg-server和显卡驱动
```bash
$ sudo pacman -S xorg-server
#nvidia显卡安装xf86-video-nouveau，Intel核心显卡安装xf86-video-intel 
$ sudo pacman -S  xf86-video-intel
```
## 字体安装
```bash
$ sudo pacman -S ttf-dejavu wqy-microhei 
```
## 安装显示管理器
LightDM 是一个跨桌面环境的显示管理器，具有代码轻量，跨桌面等特点。
```bash
$ sudo pacman -S  lightdm-gtk-greeter
$ sudo systemctl enable lightdm
```
## Cinnamon安装
正式安装cinnamon桌面
```bash
$ sudo pacman -S cinnamon
```
## 安装应用程序
　　为了便于使用，我们需要安装一些应用软件：终端，触摸板驱动、声音驱动、蓝牙，语言模块等
```bash
$ sudo pacman -S gnome-terminal xf86-input-synaptics alsa-utils
$ yaourt -S blueberry mintlocale
```
```bash
# 开机启动网络服务
$ sudo systemctl enable NetworkManager
```
　　现在重新启动系统，就可以进入我们的Cinnamon桌面啦，下一章我们进行桌面的软件安装和美化配置。