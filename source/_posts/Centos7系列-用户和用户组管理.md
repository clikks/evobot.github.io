---
title: Centos7系列:用户和用户组管理
author: Evobot
abbrlink: adac5712
date: 2018-04-02 21:30:40
categories: Centos7
tags:
  - Centos
image:
---



主要介绍Linux系统中用户和用户组的概念，以及用户和用户组文件和相应的管理命令，如`useradd`增加用户`groupadd`增加用户组等，同时介绍了如何使用`rz`、`sz`命令在Linux和Windows之间传输文件。

<!-- more -->

---

# Linux与Windows传输文件

- 在Xshell中，若需要在Windows和Centos之间传输文件，可以使用`lrzsz`软件包当中的`rz`、`sz`命令来进行上传或下载文件；
- 使用`rz`、`sz`需要安装`lrzsz`软件包，使用命令`yum -y install lrzsz`安装；
- `rz`命令是用来上传文件到Linux，上传的位置是当前目录，`rz`后面不需要跟任何参数，回车后会弹出文件对话框，选择需要上传的文件即可；
- `sz`命令用来从Linux上下载文件，使用`sz filename`的形式，回弹出保存对话框，选择Windows上保存的位置即可。

# 用户及密码配置文件

## passwd用户文件
- `passwd`文件位于`/etc/`目录下；
- `passwd`文件结构为每个用户一行，每行由冒号分割为7段:
1. 第一段为用户名;
2. 第二段为密码占位符;
3. 第三段为`UID`;
4. 第四段为`GID`;
5. 第五段为注释信息，无实际作用;
6. 第六段为家目录地址;
7. 第七段为用户shell;
- 用户shell主要为`/sbin/nologin/`,表示用户禁止登陆，以及`/bin/bash`.
- 用户创建的新用户UID和GID从1000开始。
```bash
[root@localhost ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-bus-proxy:x:999:997:systemd Bus Proxy:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:998:996:User for polkitd:/:/sbin/nologin
tss:x:59:59:Account used by the trousers package to sandbox the tcsd daemon:/dev/null:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
chrony:x:997:995::/var/lib/chrony:/sbin/nologin
lux:x:1000:1000::/home/lux:/bin/bash
clikks:x:1001:1001::/home/clikks:/bin/bash
colxu:x:1002:1002::/home/colxu:/bin/bash
```

## shadow密码文件

- `shadow`文件同样在`/etc/`目录下；
- `shadow`文件用来存储用户的密码，与`passwd`文件内容一一对应，每个用户一行，以冒号分割为9段:
1. 第一段为用户名
2. 第二段为加密的密码；
3. 第三段为上次更改密码的天数，以1970/1/1计算；
4. 第四段为密码更改限制天数，即必须满足距离上次更改密码多少天之后才能修改密码，默认为0；
5. 第五段为距离上次更改密码多少天后密码到期，必须更改；
6. 第六段为密码到期前警告期限，如提醒7天后用户密码到期；
7. 第七段为账号的失效期限，例如设置为3，则密码到期后可以继续使用3天，3天后将被锁定;
8. 第八段为距离1970/1/1的日期前账号可以使用;
9. 第九段为保留字段。
```bash
[root@localhost ~]# cat /etc/shadow
root:$6$RorrMuXF$sqkTLxh/gW4yAnb987iuYAooKBPMK9pn.zJ8JNWBwiQeNI3EccbU6dS5hvQUNb.sUA.5yAyjFkJ.ZNj4fDnTd/:17406:0:99999:7:::
bin:*:17110:0:99999:7:::
daemon:*:17110:0:99999:7:::
adm:*:17110:0:99999:7:::
lp:*:17110:0:99999:7:::
sync:*:17110:0:99999:7:::
shutdown:*:17110:0:99999:7:::
halt:*:17110:0:99999:7:::
mail:*:17110:0:99999:7:::
operator:*:17110:0:99999:7:::
games:*:17110:0:99999:7:::
ftp:*:17110:0:99999:7:::
nobody:*:17110:0:99999:7:::
systemd-bus-proxy:!!:17396::::::
systemd-network:!!:17396::::::
dbus:!!:17396::::::
polkitd:!!:17396::::::
tss:!!:17396::::::
postfix:!!:17396::::::
sshd:!!:17396::::::
chrony:!!:17396::::::
lux:!!:17462:0:99999:7:::
clikks:!!:17462:0:99999:7:::
colxu:!!:17464:0:99999:7:::
```
------------

# 用户组管理
## group用户组文件
- `group`用户组文件与`passwd`文件类似，都位于`/etc/`目录下；
- `group`文件记录了用户组的组名，以及密码，GID.

## 用户组管理命令
### 1. groupadd
- `groupadd`用于添加用户组，命令格式为`groupadd [groupname]：
```bash
[root@localhost ~]# groupadd grp1
[root@localhost ~]# tail -n1 /etc/group
grp1:x:1003:
```
- 增加用户组的同时指定GID，命令为`groupadd [-g] [GID] [Groupname]`：
```bash
[root@localhost ~]# groupadd -g 1088 grp2
[root@localhost ~]# tail -n3 /etc/group
slocate:x:21:
grp1:x:1003:
grp2:x:1088:
```

### 2. groupdel
- `groupdel`命令的作用为删除用户组，格式是`groupdel [groupname]`:
```bash
[root@localhost ~]# groupdel grp1
[root@localhost ~]# tail -n3 /etc/group
colxu:x:1002:
slocate:x:21:
grp2:x:1088:
```
- 当用户组内还存在有用户时，`groupdel`无法删除用户组：
```bash
[root@localhost ~]# groupdel colxu
groupdel：不能移除用户“colxu”的主组
```

------------


# 用户管理
## useradd增加用户
- `useradd`命令用来新增用户，`useradd`新增用户时可以指定UID,用户组&GID，家目录，用户shell：
> **-u**: 指定用户的UID；
> **-g**: 指定用户的用户组或GID；
> **-d**: 指定用户的家目录；
> **-s**：指定用户的shell；
> **-M**：表示不创建用户的家目录。
- 如下，指定用户UID,GID,家目录，SHELL：
```bash
[root@localhost ~]# useradd -u 1099 -g grp2 -d /home/mayun -s /bin/bash mahuateng
[root@localhost ~]# tail -n 3 /etc/passwd
clikks:x:1001:1001::/home/clikks:/bin/bash
colxu:x:1002:1002::/home/colxu:/bin/bash
mahuateng:x:1099:1088::/home/mayun:/bin/bash
[root@localhost ~]# ls /home/
clikks  colxu  lux  mayun
```
- `-g`选项后如果跟的是不存在的用户组，则会报错：
```bash
[root@localhost ~]# useradd -M -u 1100 -g 1032 -s /bin/bash usertest
useradd：“1032”组不存在
```

## userdel删除用户
- `userdel`命令用来删除用户，常用的为`-r`选项，用来在删除用户时同时删除用户的家目录，默认linux在删除用户时，不会删除用户的家目录：
```bash
[root@localhost ~]# useradd -d /home/user110 -u 1101 -g 1088 -s /bin/bash usertest2
[root@localhost ~]# userdel usertest2
[root@localhost ~]# ls /home/
clikks  colxu  lux  mayun  user110
[root@localhost ~]# userdel -r mahuateng
[root@localhost ~]# ls /home/
clikks  colxu  lux  user110
```
- 在新增用户时，用户的UID会跟随前一个用户的UID增加，GID并不随前一个GID增加，而是随UID增加

## usermod修改用户
- usermod可以修改已经存在的用户的UID,GID,SHELL,家目录以及增加附属组，usermod的参数选项如下：
> **-u**: 更改用户的UID;
> **-g**: 更改用户的的GID或用户组；
> **-s**: 更改用户的默认shell；
> **-d**: 更改用户的家目录；
> **-L**: 锁定用户，禁止用户登陆，同时/etc/shadow文件内，用户的密码前会增加一个`!`号；
> **-U**: 为锁定的用户解锁；
> **-G**: 增加用户的的附属组，如果`-G`后只跟一个组名，则会覆盖用户之前的附属组，添加多个附属组，需要在`-G`后一次跟随多个组名。
- `id`命令用来查看用户的UID,GID,附属组信息，格式为`id [username]`:
```bash
[root@localhost ~]# id lux
uid=1000(lux) gid=1000(lux) 组=1000(lux)
```
- 更改用户的UID,GID,shell，家目录,如下，更改用户colxu的信息：
```bash
[root@localhost ~]# tail -n 4 /etc/passwd
lux:x:1000:1000::/home/lux:/bin/bash
clikks:x:1001:1001::/home/clikks:/bin/bash
colxu:x:1002:1002::/home/colxu:/bin/bash
usertest:x:1100:1088::/home/usertest:/bin/bash
[root@localhost ~]# usermod -u 1010 -g grp2 -d /home/lux -s /sbin/nologin colxu
[root@localhost ~]# tail -n 3 /etc/passwd
clikks:x:1001:1001::/home/clikks:/bin/bash
colxu:x:1010:1088::/home/lux:/sbin/nologin
usertest:x:1100:1088::/home/usertest:/bin/bash
[root@localhost ~]# su colxu
This account is currently not available.
```
- 为colxu用户增加附属组,不论用户的附属组存在多少个，再次添加附属组都会将之前的附属组清空重新添加：
```bash
[root@localhost ~]# id colxu
uid=1010(colxu) gid=1088(grp2) 组=1088(grp2)
[root@localhost ~]# usermod -G lux colxu
[root@localhost ~]# id colxu
uid=1010(colxu) gid=1088(grp2) 组=1088(grp2),1000(lux)
[root@localhost ~]# usermod -G clikks colxu
[root@localhost ~]# id colxu
uid=1010(colxu) gid=1088(grp2) 组=1088(grp2),1001(clikks)
[root@localhost ~]# usermod -G lux,clikks,root colxu
[root@localhost ~]# id colxu
uid=1010(colxu) gid=1088(grp2) 组=1088(grp2),0(root),1000(lux),1001(clikks)
[root@localhost ~]# usermod -G mayun colxu
[root@localhost ~]# id colxu
uid=1010(colxu) gid=1088(grp2) 组=1088(grp2),1101(mayun)
```
---

