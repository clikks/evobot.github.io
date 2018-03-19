---
title: Centos7安装配置
tags:
  - Linux
  - Centos
categories:
  - Linux
author: Evobot
image: 'http://p5qynomrl.bkt.clouddn.com/1521483772581ovq9z70a.png?imageslim'
abbrlink: 79bae30d
date: 2017-08-18 22:11:00
---
在VMware虚拟机中安装Centos7，首先要创建一个虚拟机，再使用Centos7的安装光盘镜像进行安装。
<!--more-->
# 1. 创建虚拟机
下载VMware虚拟机可以到[猿课资源下载地址](http://r.aminglinux.com)下载安装
- 点击创建新的虚拟机，进入新建虚拟机向导
![新建虚拟机向导](http://p5qynomrl.bkt.clouddn.com/1521483772581ovq9z70a.png?imageslim)
- 点击下一步，按照下图选择：
![选择操作系统](http://p5qynomrl.bkt.clouddn.com/1521483882928d57huceg.png?imageslim)
- 下一步之后可以为虚拟机命名和指定虚拟机存储位置，然后根据需要为虚拟机分配CPU和内存；
- 在网络类型选择上，建议选择NAT网络地址转换，之后一路下一步，选择创建新的虚拟磁盘，并为虚拟机分配磁盘大小：
![分配磁盘](http://p5qynomrl.bkt.clouddn.com/1521484161120xv8g8slh.png?imageslim)
- 之后可以再次对虚拟机的硬件进行配置，这里不再进行配置，点击下一步到完成。
# 2.安装系统
## 2.1 安装准备
- 在新建的虚拟机窗口内点击编辑虚拟机设置，再点击CD/DVD，然后选择已经下载的Centos7的光盘镜像。
![选择光盘镜像](http://p5qynomrl.bkt.clouddn.com/1521484660545p3c3lret.png?imageslim)
- 然后点击开启此虚拟机，进行操作系统安装;
- 进入到安装窗口后，在弹出的画面中，选择**Install CentOS Linux 7**并回车两次;
![安装界面](http://p5qynomrl.bkt.clouddn.com/1521484814296webaenyu.png?imageslim)
- 之后弹出的选择语言界面，选择中文后点击继续：
![选择语言](http://p5qynomrl.bkt.clouddn.com/1521485023345lpdog8f2.png?imageslim)
- 在下面的界面中，安装源选择本地介质，软件选择最小化安装，然后点击安装位置，选择我要配置分区并点击完成：
![配置分区](http://p5qynomrl.bkt.clouddn.com/1521485274469dyyh8af6.png?imageslim)
## 2.2 系统分区
- 选择标准分区，挂载点选择**/boot**,分配容量200M，点击添加挂载点：
![boot分区](http://p5qynomrl.bkt.clouddn.com/15214855901880dtt7vj5.png?imageslim)
- 再点击加号新增一个分区，挂载点血选择**swap**，分配2048M容量并添加挂载点；磁盘分区时,swap分区一般为内存的两倍,如果内存大于4G,最大swap分区为8G,不建议继续按照两倍内存容量增加.
![swap分区](http://p5qynomrl.bkt.clouddn.com/1521485815266oe8jxf63.png?imageslim)
- 新增根分区挂载点，将剩余空间全都分配给根分区；
![根分区](http://p5qynomrl.bkt.clouddn.com/1521485944458djss9n9e.png?imageslim)
- 最后点击完成，在弹出的更改摘要窗口点击接受更改，然后点击开始安装，在安装的过程中点击ROOT密码，为root用户设置密码，之后等待系统安装完成即可。
---
