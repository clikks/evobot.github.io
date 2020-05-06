---
title: Centos7系列:用户身份切换和sudo命令
author: Evobot
abbrlink: 9f1fc446
date: 2018-04-05 02:18:26
categories: Centos7
tags:
  - Centos
image:
---



本文主要介绍`su`切换用户身份命令的用法和作用、`sudo`命令的用法和`visudo`配置sudo命令权限，以及如何禁止root用户ssh远程登录。

<!-- more -->

---

# su命令
## 切换用户身份
- su命令用来切换用户，命令格式为`su [-] [username]`,`[-]`号加与不加的区别在于是否彻底切换用户身份，如用户环境变量，家目录、配置文件等。
- `su`也可以以其他用户身份执行命令，命令为`su - -c "cmd" [username]`:
```bash
[root@localhost ~]# su - -c "touch /tmp/software.111" lux
[root@localhost ~]# ls -l /tmp/ | head -n 4
总用量 131076
-rw-------. 1 root root      1422 10月 20 23:23 anaconda-ks.cfg
-rw-r--r--. 2 root root 134217728 10月 27 21:37 image.img
-rw-rw-r--. 1 lux  lux          0 10月 30 22:48 software.111
```
- 若切换到无家目录的用户下，会出现用户无环境变量、家目录、配置文件的情况：
```bash
[root@localhost ~]# su - usertest
su: 警告：无法更改到 /home/usertest 目录: 没有那个文件或目录
-bash-4.2$ pwd
/root
-bash-4.2$ 
```

## 配置无家目录用户的登陆环境
- 给无家目录的用户创建家目录并配置登陆环境需要以下几步：
1.  创建家目录；
2.  更改目录权限；
3.  从`/etc/skel/`目录下复制`.bash`配置文件；
4.  更改配置文件权限。
```bash
[root@localhost ~]# mkdir /home/usertest
[root@localhost ~]# id usertest
uid=1100(usertest) gid=1088(grp2) 组=1088(grp2)
[root@localhost ~]# chown usertest:grp2 /home/usertest/
[root@localhost ~]# ls -ld /home/usertest/
drwxr-xr-x. 2 usertest grp2 6 10月 30 22:54 /home/usertest/
[root@localhost ~]# ls /etc/skel/.bash
.bash_logout   .bash_profile  .bashrc        
[root@localhost ~]# cp /etc/skel/.bash* /home/usertest/
[root@localhost ~]# chown -R usertest:grp2 /home/usertest/
[root@localhost ~]# ls -la /home/usertest/
总用量 12
drwxr-xr-x. 2 usertest grp2  62 10月 30 22:56 .
drwxr-xr-x. 8 root     root  88 10月 30 22:54 ..
-rw-r--r--. 1 usertest grp2  18 10月 30 22:56 .bash_logout
-rw-r--r--. 1 usertest grp2 193 10月 30 22:56 .bash_profile
-rw-r--r--. 1 usertest grp2 231 10月 30 22:56 .bashrc
[root@localhost ~]# su - usertest
上一次登录：一 10月 30 22:52:05 CST 2017pts/0 上
[usertest@localhost ~]$ 
```

------------


# sudo命令
## visudo配置
- 系统配置sudo的命令是`visudo`,`visudo`实际上修改的是`/etc/sudoers`文件，但是不建议使用vi去修改这个文件，而是使用`visudo`命令，`visudo`能够对修改的内容排错。
- `visudo`可以将用户分组，命令分组，格式如下：
```bash
#默认格式如下，添加其他用户时，只要将最后一个ALL更改为需要让用户执行的命令，命令格式必须为绝对路径
## The COMMANDS section may have other options added to it.
##
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
lux     ALL=(ALL)       /usr/bin/ls, /usr/bin/cat
```
- 命令分组采用如下的格式：
```bash
## Networking
# Cmnd_Alias NETWORKING = /sbin/route, /sbin/ifconfig, /bin/ping, /sbin/dhclient, /usr/bin/net, /sbin/iptables, /usr/bin/rfcomm, /usr/bin/wvdial, /sbin/iwconfig, /sbin/mii-tool
##Lux cmd
Cmnd_Alias LUX = /usr/bin/ls, /usr/bin/cat, /usr/bin/mv
```
- 也可以将几个用户添加到一个虚拟的用户组，再对用户组进行配置。例如：
```bash
## User Aliases
## These aren't often necessary, as you can use regular groups
## (ie, from files, LDAP, NIS, etc) in this file - just use %groupname
## rather than USERALIAS
# User_Alias ADMINS = jsmith, mikem
User_Alias SUPER = lux, colxu, clikks
```
- 也可以对系统的用户组进行配置：
```bash
## Allows people in group wheel to run all commands
%wheel  ALL=(ALL)       ALL
```

------------

# 用户root权限管理
## 禁止root远程登陆
- 禁止root远程登陆需要修改sshd的配置文件，如下所示：
```bash
#LoginGraceTime 2m
#将permitRootLogin yes改为no即可。
PermitRootLogin no
#StrictModes yes
#MaxAuthTries 6
#MaxSessions 10
```
- 然后重启`sshd`服务：
```bash
[root@localhost ~]# systemctl restart sshd.service
```
## 配置其他用户sudo权限

- 禁止root登陆后，需要对其他用户分配root的权限，在`visudo`里，给用户增加`su`命令的权限：
```bash
User_Alias SKYS = lux, colxu, clikks
##Lux cmd
Cmnd_Alias SU = /usr/bin/su
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
SKYS    ALL=(ALL)       NOPASSWD: SU
```
---

