---
title: 'Centos7系列:单用户及救援模式和虚拟机克隆操作'
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
image: 'http://p5qynomrl.bkt.clouddn.com/1521312295772z2f2ptcr.png?imageslim'
abbrlink: 19bc2f97
date: 2018-03-22 22:21:04
photo:
---

对于忘记root密码的情况，我们可以进入Centos的单用户模式或者使用安装光盘进入救援模式，来修改我们的root密码，下面为如何进入单用户模式和救援模式并修改root密码的详细步骤。另外介绍了如何在VMware中克隆虚拟机并在Linux中使用SSH相互登陆。
<!-- more -->

# Centos7单用户模式

## 配置单用户模式

1. 在Centos7启动界面下，对第一个启动项按**e**键进入配置界面。
  ![grub配置](http://p5qynomrl.bkt.clouddn.com/1521312295772z2f2ptcr.png?imageslim)
2. 将光标定位到`linux16`开头的行，再将光标移动到**ro**位置。
  ![启动项配置](http://p5qynomrl.bkt.clouddn.com/15213123237461a612knk.png?imageslim)
3. 将**ro**只读修改为**rw**读写模式，并且添加`init=/sysroot/bin/sh`在**rw**后面。
  ![修改启动项](http://p5qynomrl.bkt.clouddn.com/1521312341229joes7584.png?imageslim)
4. **sysroot**就是我们原先的系统root路径，完成之后，按**Ctrl+x**键，保存退出。

## 进入单用户模式

- 进入到系统后，需要切换到我们原有系统的环境下再继续操作，所以首先执行`chroot /sysroot/`命令切换到我们原系统**root**环境。

  ```bash
  :/# chroot /sysroot/
  ```

## 修改系统语言编码

- 为了防止出现乱码的情况，首先修改系统语言环境为英文。

  ```bash
  :/# LANG=en
  ```

## 修改root密码

- 这时候我们执行`passwd root`来修改root的密码。

  ```bash
  :/# passwd root
  Changing password for user root.
  New password:
  Retype new password:
  ```

## 创建autorelabel文件

- 修改完密码后，还需要创建**autorelabel**文件，这样重启才能进入系统。

  ```bash
  :/# touch /.autorelabel
  ```

- 执行完所有操作后重启计算机即可使用新的密码登陆root账户。

# Centos7救援模式

## 系统运行级别

- centos一共有0~6这7个运行级别，没有图形界面的情况下，默认运行级别为3，下面的命令可以查看系统的所有运行级别：

  ```bash
  [root@centos ~]# ls -l /usr/lib/systemd/system/runlevel*target
  lrwxrwxrwx 1 root root 15 Mar 12 16:48 /usr/lib/systemd/system/runlevel0.target -> poweroff.target
  lrwxrwxrwx 1 root root 13 Mar 12 16:48 /usr/lib/systemd/system/runlevel1.target -> rescue.target
  lrwxrwxrwx 1 root root 17 Mar 12 16:48 /usr/lib/systemd/system/runlevel2.target -> multi-user.target
  lrwxrwxrwx 1 root root 17 Mar 12 16:48 /usr/lib/systemd/system/runlevel3.target -> multi-user.target
  lrwxrwxrwx 1 root root 17 Mar 12 16:48 /usr/lib/systemd/system/runlevel4.target -> multi-user.target
  lrwxrwxrwx 1 root root 16 Mar 12 16:48 /usr/lib/systemd/system/runlevel5.target -> graphical.target
  lrwxrwxrwx 1 root root 13 Mar 12 16:48 /usr/lib/systemd/system/runlevel6.target -> reboot.target
  ```

- 其中的`runlevel`后面的数字就是对应的运行级别，在Centos7中，单用户模式对应的其实是`rescue.target`；

- 7个运行级别所对应的模式如下表：

|    运行级别     |    对应target     |                对应模式                |
| :-------------: | :---------------: | :------------------------------------: |
|    runlevel0    |  poweroff.target  |                  关机                  |
|    runlevel1    |   rescue.target   |               单用户模式               |
| runlevel2、3、4 | multi-user.target | 多用户模式，无图形界面 |
|    runlevel5    | graphical.target  |                图形界面                |
|    runlevel6    |   reboot.target   |                  重启                  |


## Centos7救援模式

- 有些主机为grub进行了加密，这样在忘记了密码的情况下，就无法进入单用户模式；而救援模式则是使用安装系统时的安装光盘来进行修改密码的操作。

- 将虚拟机关机，设置光驱为安装系统时的安装光盘，选择VMware菜单中的虚拟机-电源-打开电源时进入固件；

- 在BIOS中选择boot，使用**+-**号将CD-ROM移到首位，再按**F10**保存退出；

  ![bios设置](http://p5qynomrl.bkt.clouddn.com/152173531953668h6dyus.png?imageslim)

- 进入安装光盘的启动界面，选择第三项`Troubleshooting`，然后选择`Rescue a CentOS system`回车两次；

- 然后进入交互界面，输入1表示`Continue`回车，然后再次回车进入shell；

- 这时会有提示`chroot /mnt/sysimage`，这个`sysimage`就是我们原系统的根目录，执行这条命令的作用与单用户模式的`chroot /sysroot/`相同；

- `chroot`之后就会进入bash，然后使用`passwd`命令更改root密码并重启系统即可，重启系统时，需要修改BIOS启动项为`Hard Drive`或者将光驱镜像取消。

# 虚拟机克隆

虚拟机克隆可以让我们在不用创建新虚拟机的情况下，从原虚拟机克隆出一个新的虚拟机来使用。

- 关闭虚拟机，选择VMware菜单上的虚拟机-管理-克隆，进入克隆虚拟机向导界面；

- 选择克隆自虚拟机中的当前状态-创建链接克隆，然后为克隆的虚拟机命名并指定存储位置；

- 启动克隆出来的虚拟机，登陆系统，这时系统的IP地址与原虚拟机是相同的，所以我们需要修改系统的IP地址，并删除配置文件中的UUID行，然后重启网络服务；

- 之后修改系统的主机名，使用`hostname`命令可以查看系统的主机名，使用`hostnamectl set-hostname evobot02`来设置新的主机名，然后退出重新登陆，新的主机名即可生效：

  ```bash
  [root@localhost ~]# hostname
  localhost.localdomain
  [root@localhost ~]# hostnamectl set-hostname evobot02
  [root@localhost ~]# hostname
  evobot02
  [root@evobot02 ~]# logout
  [root@evobot02 ~]#
  ```

- 系统的主机名配置文件为`/etc/hostname`。


# Linux相互登陆

在实际使用中，经常需要从一台Linux服务器登陆到另一台Linux服务器，这时候就需要掌握如何在两台Linux之间相互登陆的知识。

##  ssh命令登陆

- 命令`w`可以查看有哪些客户端登陆到系统：

  ```bash
  [root@evobot02 ~]# w
   01:15:29 up 26 min,  2 users,  load average: 0.00, 0.01, 0.02
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  root     tty1                      01:07    8:25   0.01s  0.01s -bash
  root     pts/0    192.168.253.1    01:07    1.00s  0.05s  0.03s w
  ```

- 使用命令`ssh ip`可以登陆另一个Linux系统：

  ```bash
  [root@evobot02 ~]# ssh 192.168.253.133
  The authenticity of host '192.168.253.133 (192.168.253.133)' can't be established.
  ECDSA key fingerprint is SHA256:kXUH0kDjliChKsWQTmWL/uDammTO8OYDHj4CaQE4oYw.
  ECDSA key fingerprint is MD5:e5:87:58:2d:d9:a1:38:12:b6:36:a0:41:d2:b9:50:ab.
  Are you sure you want to continue connecting (yes/no)? yes
  Last login: Fri Mar 23 01:21:14 2018 from 192.168.253.134
  [root@evobot01 ~]#
  [root@evobot01 ~]# w
   01:22:59 up 28 min,  2 users,  load average: 0.00, 0.01, 0.05
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  root     pts/0    192.168.253.134  01:22    3.00s  0.05s  0.03s w
  root     pts/1    192.168.253.1    01:14    7:31   0.03s  0.03s -bash
  ```

- 可以看到主机名已经变成了evobot01，`w`命令看到有从IP`192.168.253.134`登陆的用户；

- `ssh`命令的标准写法是`ssh username@ip -p port`，在不指定username的情况下，是使用当前系统的用户名去连接对端主机，`-p`参数是指定ssh的端口：

  ```bash
  [root@evobot01 ~]# ssh root@192.168.253.134 -p 22
  The authenticity of host '192.168.253.134 (192.168.253.134)' can't be established.
  ECDSA key fingerprint is SHA256:kXUH0kDjliChKsWQTmWL/uDammTO8OYDHj4CaQE4oYw.
  ECDSA key fingerprint is MD5:e5:87:58:2d:d9:a1:38:12:b6:36:a0:41:d2:b9:50:ab.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added '192.168.253.134' (ECDSA) to the list of known hosts.
  root@192.168.253.134's password:
  Last login: Fri Mar 23 01:07:25 2018 from 192.168.253.1
  ```

## 密钥登陆

两台Linux之间也可以使用密钥登陆，首先需要生成主机的ssh密钥对，然后将公钥添加到对端机器的`authorized_keys`文件。

- 生成密钥对，使用`ssh-keygen`命令：

  ```bash
  [root@evobot01 ~]# ssh-keygen
  Generating public/private rsa key pair.
  Enter file in which to save the key (/root/.ssh/id_rsa):# 指定存储密钥的路径，回车即可
  Enter passphrase (empty for no passphrase):		# 设置密钥密码，为空即可
  Enter same passphrase again:	# 再次输入密码
  Your identification has been saved in /root/.ssh/id_rsa.	# 私钥
  Your public key has been saved in /root/.ssh/id_rsa.pub.	# 公钥
  The key fingerprint is:
  SHA256:Ccn7HtJMEgDJvcPNRCZoC9vxp+91r65zq4+r7S4Iaoo root@evobot01
  The key's randomart image is:
  +---[RSA 2048]----+
  | ..=o.o          |
  |. * .=..         |
  | = = ==          |
  |. o = ++ .       |
  |     +o S        |
  |  . .  *         |
  | . . o. * .      |
  |o.  . o=.+..     |
  |E    .o=XO=o.    |
  +----[SHA256]-----+
  ```

- 生成的密钥对默认在家目录的`.ssh/`目录下，然后将公钥的内容写入对端机器`authorized_keys`;

- 关闭selinux后，就可以使用密钥认证登陆对端主机：

  ```bash
  [root@evobot01 ~]# cat .ssh/id_rsa.pub
  ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDsHTIbFZs6WtQzBXZCwul5fqb7pVGcISVnzD69MmsDIyTnPL3F7M0Rl/a6nQcYC+G3s+OR+d+hjNrOcceTX+odZdCnzcRkALwS1r45KcxGpic7m9vMh3Kj04hhTbhCmZJTm9TTSSjhOyF/hNhKavIOfUPJdYy6SS7zAO3GLr67rwg8tmL/qf3muVvhdAXg2lZh0xQXJa7IB/XBq4zYb32sQs6RGE2Sf4NnzTC6IpSYUWfucMJCaZeevCraYe7fo3kuh8fa7kWj+qwbuBLHojzfcEro7elpur4qh+OdNHCeR+gcpC51qrtyvYj9JtRRKoHq1ZG6VJRRFSV+Dvo9vSnX root@evobot01
  [root@evobot01 ~]# ssh root@192.168.253.134 -p 22
  Last login: Fri Mar 23 01:30:56 2018 from 192.168.253.1
  ```

---

