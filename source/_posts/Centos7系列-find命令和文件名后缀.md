---
title: Centos7系列:find命令和文件名后缀
author: Evobot
abbrlink: f4c9d6be
date: 2018-03-30 21:13:03
categories: Centos7
tags: [Linux, Centos]
image:
---



**find**命令是Linux下的搜索命令，功能非常强大，能够进行多条件精确搜索，本文主要介绍**find**命令的常用选项和用法，并介绍Linux下的文件名后缀相关知识。

<!-- more -->

---

# 搜索命令

## locate搜索
- 默认centos未带有`locate`命令，需要安装**mlocate**软件包，`yum install -y mlocate`
- 安装完成后，首次使用需要更新数据库，否则提示如下：
```bash
[root@localhost ~]# locate ls
locate: 无法执行 stat () `/var/lib/mlocate/mlocate.db': 没有那个文件或目录
```
- 更新数据库使用`updatedb`命令，搜索使用`locate [keyword]`即可搜索。

------------

## find搜索
### find的命令语法
- find基础语法为`find [path] -type [d|f|b|c|s|l] -name [filename|dirname]`,**-type**.**-name**都是find命令的参数之一，在使用中可以省略一些参数，**-name**中文件名或目录名支持通配。
- 文件的type类型**'d'**代表目录，**'l'**代表软链接，**'f'**代表文件，**'b'**代表块设备，**'s'**代表socket文件，**'c'**代表字符串设备.
```bash
[root@localhost ~]# find /dev/ -type b
/dev/sda3
/dev/sda2
/dev/sda1
/dev/sda
/dev/sr0
```
### find命令时间参数
- `stat`命令用来查看文件的详细信息，包括块大小，访问时间等；更改系统语言使用`LANG=en`。

#### **ctime**状态时间
- `ctime`(Change Time)记录文件权限，所有者、所属组，inode，文件大小等文件属性最后一次被修改的时间，单位为天。
```bash
[root@localhost ~]# chown :lux files.ini 
[root@localhost ~]# stat files.ini 
  File: 'files.ini'
  Size: 80            Blocks: 8          IO Block: 4096   regular file
Device: 803h/2051d    Inode: 67157950    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: ( 1000/     lux)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2017-10-25 23:10:21.886836753 +0800
Modify: 2017-10-27 22:46:28.963602514 +0800
Change: 2017-10-27 23:00:52.041127046 +0800
 Birth: -
```
- find中`ctime`参数使用形式为`-ctime [+-][n]`，减号为`ctime`在n天以内，加号为`ctime`在n天以前：
```bash
[root@localhost ~]# find . -type f -ctime -1 
./.bash_history
./files.ini
./image.img
./virtual.img
[root@localhost ~]# ll ./image.img
-rw-r--r--. 2 root root 134217728 Oct 27 21:37 ./image.img
[root@localhost ~]# date
Fri Oct 27 23:20:32 CST 2017
```

#### **mtime**修改时间
- `mtime`(Modity Time)记录最后一次对文件内容进行增加删除的时间，单位为天。
```bash
[root@localhost ~]# echo "1234" >> files.ini 
[root@localhost ~]# stat files.ini 
  File: 'files.ini'
  Size: 80            Blocks: 8          IO Block: 4096   regular file
Device: 803h/2051d    Inode: 67157950    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2017-10-25 23:10:21.886836753 +0800
Modify: 2017-10-27 22:46:28.963602514 +0800
Change: 2017-10-27 22:46:28.963602514 +0800
 Birth: -
```
- 更改`mtime`,则`ctime`一定会变，因为增删内容导致文件大小变化，使`ctime`时间变化。
- find中`mtime`和`ctime`的用法一致，查看`mtime`在n天以前或n天以内:
```bash
[root@localhost ~]# find /etc/ -type f -mtime +5 -name "*.sh"
/etc/profile.d/colorgrep.sh
/etc/profile.d/which2.sh
/etc/profile.d/less.sh
/etc/profile.d/colorls.sh
/etc/profile.d/256term.sh
/etc/profile.d/lang.sh
/etc/dhcp/dhclient.d/chrony.sh
/etc/kernel/postinst.d/51-dracut-rescue-postinst.sh
[root@localhost ~]# ll /etc/profile.d/less.sh
-rw-r--r--. 1 root root 121 Jul 31  2015 /etc/profile.d/less.sh
[root@localhost ~]# date
Fri Oct 27 23:31:31 CST 2017
```

#### **atime**访问时间
- `atime`(Access Time)记录文件内容最后一次被查看的时间，单位为天。
```bash
[root@localhost ~]# cat files.ini 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
1234
1234
1234
[root@localhost ~]# stat files.ini 
  File: 'files.ini'
  Size: 80            Blocks: 8          IO Block: 4096   regular file
Device: 803h/2051d    Inode: 67157950    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: ( 1000/     lux)
Context: unconfined_u:object_r:admin_home_t:s0
Access: 2017-10-27 23:08:40.668946564 +0800
Modify: 2017-10-27 22:46:28.963602514 +0800
Change: 2017-10-27 23:00:52.041127046 +0800
 Birth: -
```
- `atime`同样和`mtime`、`ctime`使用方法相同：
```bash
[root@localhost ~]# find . -type f -atime -1 -name "*.ini"
./files.ini
[root@localhost ~]# ll files.ini 
-rw-r--r--. 1 root lux 80 Oct 27 22:46 files.ini
```
#### **mmin**修改时间
- `mmin`参数用来查找n分钟之前或n分钟之内的文件，使用方法同`mtime`等相同,单位为分钟。
```bash
[root@localhost ~]# find . -mmin -120 -type f 
./files.ini
```
### find其他参数
#### inum按inode查找
- `inum`参数用来依据inode号查找文件，比如有多个inode号相同的文件，使用`-inum [inode]`.
```bash
[root@localhost ~]# ls -i virtual.img 
75 virtual.img
[root@localhost ~]# find / -inum 75 -type f -mtime -1
/root/virtual.img
/tmp/image.img
```
#### size按文件大小查找
- `-size`参数可以制定需要查找的文件大小，如`-size -10K``-size +10M`:
```bash
#查找小于1b的文件
[root@localhost ~]# find . -size -1b -type f 
./dirname/firstdir/file1.txt
./dirname/.files.txt.swp
./dirname/.files.txt.swx
./dirname/files.txt
./dirname/.file2.txt
./dirname/file2.txt
[root@localhost ~]# ll ./dirname/firstdir/file1.txt
-rw-r--r--. 1 root lux 0 Oct 25 23:21 ./dirname/firstdir/file1.txt
```
#### 'o'条件或
- `-o`参数可以让find的搜索条件使用或，即搜索条件满足其中一个即可：
```bash
[root@localhost ~]# find /tmp/ -size +10M -o -mtime -1 -o -type f -o -name "image*" 
/tmp/
/tmp/anaconda-ks.cfg
/tmp/file.txt
/tmp/image.img
```

### find的exec选项
- `exec`选项可以对find搜索出来的结果做进一步的处理，例如对搜索结果执行`mv`、`ls`等命令.
- `exec`的使用方法，`find [path] -[arg] -[exec] [command] {} \;`，其中`{}`代表find搜索出来的结果，命令结尾的`\;`之后可以继续添加`-exec`选项。
```bash
[root@localhost ~]# find . -mmin -120 -type f -exec ls -l {} \;
-rw-r--r--. 1 root lux 80 Oct 27 22:46 ./files.ini
[root@localhost ~]# mv virtual.img.bak virtual.img
[root@localhost ~]# find . -size +10M -type f -exec ls -l {} \; -exec mv {} {}.bak \; -exec ls -l \;
-rw-r--r--. 2 root root 134217728 Oct 27 21:37 ./virtual.img
total 131084
-rw-------. 1 root root      1422 Aug 19 07:25 anaconda-ks.cfg
-rw-r--r--. 1 root lux         80 Oct 27 22:46 files.ini
-rw-r--r--. 2 root root 134217728 Oct 27 21:37 virtual.img.bak
```
# 文件名后缀

- 在我们使用过程中，创建的文件经常会带后缀名，比如`.txt`、`.sh`等；

- 但实际上在Linux中，文件的类型并不是依靠后缀名区分，之所以添加后缀名，主要是方便用户区分；

- 想要准确判断一个文件的类型，可以使用`file`命令：

  ```bash
  [root@localhost ~]# file install.log
  install.log: UTF-8 Unicode text

  [root@localhost ~]# ls -l /var/mail
  lrwxrwxrwx 1 root root 10 08-13 00:11 /var/mail -> spool/mail

  [root@localhost ~]# file /var/mail
  /var/mail: symbolic link to `spool/mail'
  ```

  ---