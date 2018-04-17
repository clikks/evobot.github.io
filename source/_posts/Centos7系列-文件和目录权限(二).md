---
title: 'Centos7系列:文件和目录权限(二)'
author: Evobot
abbrlink: a0538766
date: 2018-03-29 23:33:22
categories: Centos7
tags: [Linux, Centos]
image:
---



主要介绍了Linux特殊权限**set_uid**、**set_gid**和**stick_bit**的用法和作用，以及软链接和硬链接的区别和命令的用法。

<!-- more -->

---

# 特殊权限
## **set_uid**
- `set_uid`使用于二进制可执行文件，可以让其他用户在执行文件时，拥有文件所有者的权限，例如`passwd`命令：
```bash
[root@localhost ~]# ls -l /usr/bin/passwd 
-rwsr-xr-x. 1 root root 27832 6月  10 2014 /usr/bin/passwd
```
- 所有者原来的"x"权限位上的"s"即是`set_uid`权限，让文件具有`set_uid`权限的命令为`chmod u+s filename`,例如下面让`ls`命令具有`set_uid`权限:
```bash
[lux@localhost ~]$ ls /root/
ls: 无法打开目录/root/: 权限不够
[root@localhost ~]# ls -l /usr/bin/ls
-rwxr-xr-x. 1 root root 117656 11月  6 2016 /usr/bin/ls
[root@localhost ~]# chmod u+s !$
chmod u+s /usr/bin/ls
[root@localhost ~]# ls -l /usr/bin/ls
-rwsr-xr-x. 1 root root 117656 11月  6 2016 /usr/bin/ls
[lux@localhost ~]$ ls /root/
anaconda-ks.cfg  dirname  files.ini  xxx.ini
```
- 另一种方式是`chmod u=rws filename`命令，这样得到的**set_uid**权限为大写的"S"，是因为修改权限时，没有赋予"x"权限，但是并不影响命令的执行效果，因为"S"本身具有执行权限，并且文件的执行所使用的权限在文件的其他人权限上具有"x"权限。若需要变成小写"s",增加"x"权限即可：`chmod u+x filename`
```bash
[root@localhost ~]# chmod u=rws /usr/bin/ls
[root@localhost ~]# ls -l /usr/bin/ls
-rwSr-xr-x. 1 root root 117656 11月  6 2016 /usr/bin/ls
```
- 删除文件的**set_uid**权限，使用命令`chmod u-s filename`

## **set_gid**
- **set_gid**用来给文件(二进制可执行)或者目录增加"s"权限，使得其他用户在使用文件或者目录时，具有文件、目录所属组的权限。
- 当目录被设置了**set_gid**时，任何用户在目录下创建文件或者目录都和父目录具有相同的组。
- 给文件或者目录增加**set_uid**权限，使用命令`chmod g+s [files|dir]`,例如：
```bash
[root@localhost ~]# chmod g+s dirname/
[root@localhost ~]# ls -dl dirname/
drwxr-sr-x. 3 root lux 101 10月 25 23:23 dirname/
[root@localhost ~]# ls -l dirname/
总用量 0
-rw-r--r--. 1 root lux  0 10月 25 23:17 files.txt
drwxr-xr-x. 3 root lux 37 10月 25 23:21 firstdir
[root@localhost ~]# cd dirname/
[root@localhost dirname]# touch file2.txt
[root@localhost dirname]# ls -l file2.txt 
-rw-r--r--. 1 root lux 0 10月 26 22:59 file2.txt
[root@localhost dirname]# mkdir dir2
[root@localhost dirname]# ls -dl dir2/
drwxr-sr-x. 2 root lux 6 10月 26 23:04 dir2/
```
- **set_gid**作用于文件时，与**set_uid**功能一样，其他用户对文件具有文件所属组的权限。
- 删除**set_git**权限使用`chmod g-s [file|dir]`。

## **stick_bit**
- **stick_bit**是防删除位，在linux系统中，tmp目录就具有**stick_bit**权限，是指除root外，其他用户无法删除目录下的文件。
- 因为文件是否能够被删除，取决于文件所在的目录是否具有写权限，所以**stick_bit**作用于目录，使用`chmod o+t dirname`来增加权限，
```bash
[root@localhost /]# ls -ld /tmp/
drwxrwxrwt. 9 root root 276 10月 20 23:23 /tmp/
[root@localhost /]# cd /tmp/
[root@localhost tmp]# ls -l
总用量 4
-rw-------. 1 root root 1422 10月 20 23:23 anaconda-ks.cfg
drwx------. 3 root root   17 10月 20 22:56 systemd-private-b843e5ed2c5a4150a3066f84f601a287-vmtoolsd.service-gmqqZM
drwx------. 3 root root   17 8月  25 22:42 systemd-private-f71033e5617942ef84a31b21fe152372-vmtoolsd.service-p491Gw
[root@localhost tmp]# su lux
[lux@localhost tmp]$ touch file.txt
[lux@localhost tmp]$ ll
总用量 4
-rw-------. 1 root root 1422 10月 20 23:23 anaconda-ks.cfg
-rw-rw-r--. 1 lux  lux     0 10月 26 23:21 file.txt
drwx------. 3 root root   17 10月 20 22:56 systemd-private-b843e5ed2c5a4150a3066f84f601a287-vmtoolsd.service-gmqqZM
drwx------. 3 root root   17 8月  25 22:42 systemd-private-f71033e5617942ef84a31b21fe152372-vmtoolsd.service-p491Gw
[lux@localhost tmp]$ exit
exit
[root@localhost tmp]# su colxu
[colxu@localhost tmp]$ ll
总用量 4
-rw-------. 1 root root 1422 10月 20 23:23 anaconda-ks.cfg
-rw-rw-r--. 1 lux  lux     0 10月 26 23:21 file.txt
drwx------. 3 root root   17 10月 20 22:56 systemd-private-b843e5ed2c5a4150a3066f84f601a287-vmtoolsd.service-gmqqZM
drwx------. 3 root root   17 8月  25 22:42 systemd-private-f71033e5617942ef84a31b21fe152372-vmtoolsd.service-p491Gw
[colxu@localhost tmp]$ rm -rf file.txt 
rm: 无法删除"file.txt": 不允许的操作
```
---

# 链接文件

## 软连接文件
- 软连接的含义相当于快捷方式，是将一个链接文件，指向所链接的实际文件上，同样对于目录，也可以创建软链接文件。
- 创建软链接使用`ln -s srcfile targetfile`命令.
```bash
[root@localhost ~]# ls -l /usr/bin/python
lrwxrwxrwx. 1 root root 7 8月  19 07:20 /usr/bin/python -> python2
```
- 创建软链接一定要使用绝对路径，否则当软链接文件被移动或复制后，会无法找到源文件。
```bash
[root@localhost ~]# ln -s /var/log/yum.log /root/yumlog
[root@localhost ~]# ll yumlog 
lrwxrwxrwx. 1 root root 16 10月 27 21:20 yumlog -> /var/log/yum.log
```


## 硬链接文件
- 硬链接只支持为文件创建链接，无法为目录创建链接
- 硬链接的特性是创建一个inode号与文件相同的文件，两个文件相互为硬链接，并且删除一个硬链接文件，并不影响其他相同inode号硬链接文件的使用。
- 硬链接并不会多占用磁盘空间，因为其inode号是同一个，inode记录了文件的所有信息。
- 创建硬链接文件，不能跨磁盘分区创建。因为分区之间拥有各自独立的inode号，两个分区会存在同样的inode号。
- 使用`ln srcfile targetfile`来创建硬链接文件。
```bash
[root@localhost ~]# dd if=/dev/zero of=/root/image.img bs=1M count=128
记录了128+0 的读入
记录了128+0 的写出
134217728字节(134 MB)已复制，3.59043 秒，37.4 MB/秒
[root@localhost ~]# ll image.img 
-rw-r--r--. 1 root root 134217728 10月 27 21:37 image.img
[root@localhost ~]# ln image.img virtual.img
[root@localhost ~]# ll image.img virtual.img 
-rw-r--r--. 2 root root 134217728 10月 27 21:37 image.img
-rw-r--r--. 2 root root 134217728 10月 27 21:37 virtual.img
[root@localhost ~]# ls -i image.img virtual.img 
75 image.img  75 virtual.img
[root@localhost ~]# ln /boot/initrd-plymouth.img  boot.img
ln: 无法创建硬链接"boot.img" => "/boot/initrd-plymouth.img": 无效的跨设备连接
```
---

