---
title: windows10 & Arch 双系统安装指南【三】
tags:
  - archlinux
  - cinnamon
  - windows10
categories: Linux
abbrlink: 42decf85
date: 2016-02-14 22:37:07
---
![](https://s1.ax1x.com/2020/04/28/JIGFbV.png)
经过上一篇的配置，我们已经有了一个比较粗糙的Cinnamon桌面的Arch系统，但是还缺少很多基础的应用软件，桌面也不够美观，这一篇我们安装常用的应用软件。
<!-- more -->

---
# 应用软件安装
## 浏览器
![](https://s1.ax1x.com/2020/04/28/JIGeC4.png)

　　Linux上常用的浏览器有**google-chrome**和**firefox**，我们就装这两个浏览器就足够了,同时还需要安装**flash**插件：
```bash
$ sudo pacman -S google-chrome firefox firefox flashplugin
```
## 输入法
　　输入法比较好用的有搜狗拼音和google拼音，我安装的是搜狗拼音：
```bash
$ sudo pacman -S fcitx-im fcitx-configtool
#然后在家目录的.xprofile文件(没有就创建)写入以下内容
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=“@im=fcitx”
#安装搜狗拼音
yaourt -S fcitx-sogoupinyin
```
注销重新登陆后就可以使用输入法了。
## 文本编辑
![](https://s1.ax1x.com/2020/04/28/JIGn29.png)

图形界面的文本编辑器我们使用**gedit**，当然也可以使用GVIM。
```bash
$ sudo pacman -S gedit gedit-plugins
#安装vim
$ sudo pacman -S vim
```
## 办公软件
![](https://s1.ax1x.com/2020/04/28/JIGuvR.png)

office我们使用wps office：
```bash
yaourt -S wps-office
```
wps会需要一些windows字体，可以从windows上复制过来。
## 图片查看
```bash
$ sudo pacman -S gpicview
```
## 下载工具
![](https://s1.ax1x.com/2020/04/28/JIGQDx.png)
```bash
$ sudo pacman -S uget aria2
```
## 压缩软件
```bash
$ sudo pacman -S file-roller unrar p7zip unzip
```
## 虚拟机
![](https://s1.ax1x.com/2020/04/28/JIGlb6.png)

Linux上的虚拟机软件有很多，这里安装的是virtualbox：
```bash
$ sudo pacman -S virtualbox
#运行虚拟机之前需要加载模块
$ sudo modprobe vboxdrv
#也可以开机自动加载模块
echo "/sbin/rcvboxdrv setup" >> .profile
```
## 分区管理
![](https://s1.ax1x.com/2020/04/28/JIGG5D.png)
```bash
$ sudo pacman -S gparted
```
## 截图软件
　　截图软件我使用的是深度截图
```bash
yaourt -S deepin-screenshot
```
## 影音软件

![](https://s1.ax1x.com/2020/04/28/JIGN2d.png)
![](https://s1.ax1x.com/2020/04/28/JIGUxA.png)

同样我使用的音乐播放器和电影播放器都是深度系列：
```bash
yaourt -S deepin-music deepin-movie
```
## 邮件软件
![](https://s1.ax1x.com/2020/04/28/JIGdKI.png)

Thunderbird是一个强大的邮件客户端，可以方便的管理我们的邮件
```bash
$ sudo pacman -S thunderbird
```
基本上常用的软件就这么多，如果自己有其他的软件需求，可以直接使用**pacman**或者**yaourt**搜索安装。

---

