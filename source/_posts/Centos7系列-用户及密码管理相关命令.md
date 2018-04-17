---
title: 'Centos7系列:用户及密码管理相关命令'
author: Evobot
abbrlink: 2af55eed
date: 2018-04-03 22:44:32
categories: Centos7
tags: [Linux, Centos]
photo: http://p5qynomrl.bkt.clouddn.com/1522771004764cu3a7i8u.png?imageslim
---


主要介绍`usermod`更改用户属性的命令的用法，和用户密码管理的相关知识，以及`mkpasswd`命令的用法和作用。

<!--more-->

----

# usermod命令

- `usermod`命令可以更改用户的**uid**和**gid**以及用户的家目录和shell；

- 在之前的`useradd`命令中，选项`useradd -G`可以为用户添加扩展组，即用户的**gid**只有一个，但可以同时也属于其他用户组，这些用户组就是用户的扩展组；

- `usermod`同样有`-G`选项，为已经存在用户添加扩展组，具体用法为`usermod -G (组名) (用户名)`：

  ```bash
  [root@evobot ~]# usermod -G evobot lux
  [root@evobot ~]# id lux
  uid=1001(lux) gid=1001(lux) 组=1001(lux),1002(evobot)
  [root@evobot ~]# usermod -G grp1 lux
  [root@evobot ~]# id lux
  uid=1001(lux) gid=1001(lux) 组=1001(lux),1003(grp1)
  ```

- 上面的例子也可以看出，当使用`-G`选项为用户添加新的扩展组时，会替换之前的扩展组，若需要添加多个扩展组，需要一次添加多个扩展组，多个组名之间使用`,`分隔：

  ```bash
  [root@evobot ~]# usermod -G evobot,grp1 lux
  [root@evobot ~]# id lux
  uid=1001(lux) gid=1001(lux) 组=1001(lux),1002(evobot),1003(grp1)
  ```

# 用户密码管理

## 更改密码

- 用户密码管理，常用的命令时`passwd`命令，由于`passwd`命令具有`ser_uid`权限，所以普通用户也可以使用该命令更改密码；

- 更改当前用户密码，直接执行`passwd`命令即可，若更改其他用户密码，则需要root用户执行`passwd`命令，用法为`passwd (username)`:

  ```bash
  [root@evobot ~]# passwd evobot 
  更改用户 evobot 的密码 。
  新的 密码：
  重新输入新的 密码：
  passwd：所有的身份验证令牌已经成功更新。

  [root@evobot ~]# tail /etc/shadow
  ntp:!!:17540::::::
  postfix:!!:17540::::::
  chrony:!!:17540::::::
  sshd:!!:17540::::::
  tcpdump:!!:17540::::::
  syslog:!!:17598::::::
  centos:!!:17602:0:99999:7:::
  lux:$1$AFvb7KAB$qg0IKBSFsGRDAO5XPaoYr/:17604:0:99999:7:::
  nginx:!!:17604::::::
  evobot:$1$ORddLa4O$eScmBtXm8sIosft7XETeO1:17624:0:99999:7:::
  ```

- 为用户更改密码后，在`/etc/shadow`文件中就可以看到用户的密码位多了加密的字符串，而没有密码的用户，密码位是两个`!`；

- 在`/etc/shadow`密码文件中，还有一些用户的密码位是`*`号，这表示用户的密码是无法使用、被锁定的：

  ```bash
  [root@evobot ~]# head /etc/shadow
  root:$1$MNOcZHup$MrsJJ3KwxTERWrlgmpjcC0:17602:0:99999:7:::
  bin:*:17110:0:99999:7:::
  daemon:*:17110:0:99999:7:::
  adm:*:17110:0:99999:7:::
  lp:*:17110:0:99999:7:::
  sync:*:17110:0:99999:7:::
  shutdown:*:17110:0:99999:7:::
  halt:*:17110:0:99999:7:::
  mail:*:17110:0:99999:7:::
  operator:*:17110:0:99999:7:::
  ```


## 锁定账户

- `passwd`的`-l`选项，可以锁定用户，用户被锁定后，在密码文件`/etc/shadow`中，密码位字符串前面会加上`!!`：

  ```bash
  [root@evobot ~]# passwd -l evobot 
  锁定用户 evobot 的密码 。
  passwd: 操作成功
  [root@evobot ~]# tail /etc/shadow
  ntp:!!:17540::::::
  postfix:!!:17540::::::
  chrony:!!:17540::::::
  sshd:!!:17540::::::
  tcpdump:!!:17540::::::
  syslog:!!:17598::::::
  centos:!!:17602:0:99999:7:::
  lux:$1$AFvb7KAB$qg0IKBSFsGRDAO5XPaoYr/:17604:0:99999:7:::
  nginx:!!:17604::::::
  evobot:!!$1$ORddLa4O$eScmBtXm8sIosft7XETeO1:17624:0:99999:7:::
  ```

- 解锁用户账户，则使用`passwd`的`-u`选项：

  ```bash
  [root@evobot ~]# passwd -u evobot 
  解锁用户 evobot 的密码。
  passwd: 操作成功
  [root@evobot ~]# tail /etc/shadow
  ntp:!!:17540::::::
  postfix:!!:17540::::::
  chrony:!!:17540::::::
  sshd:!!:17540::::::
  tcpdump:!!:17540::::::
  syslog:!!:17598::::::
  centos:!!:17602:0:99999:7:::
  lux:$1$AFvb7KAB$qg0IKBSFsGRDAO5XPaoYr/:17604:0:99999:7:::
  nginx:!!:17604::::::
  evobot:$1$ORddLa4O$eScmBtXm8sIosft7XETeO1:17624:0:99999:7:::
  ```

- 锁定用户账户还可以使用`usermod -L (username)`命令，用户被锁定后，在`/etc/shadow`中的密码字符串前面会有一个`!`：

  ```bash
  [root@evobot ~]# usermod -L evobot
  [root@evobot ~]# tail /etc/shadow
  ntp:!!:17540::::::
  postfix:!!:17540::::::
  chrony:!!:17540::::::
  sshd:!!:17540::::::
  tcpdump:!!:17540::::::
  syslog:!!:17598::::::
  centos:!!:17602:0:99999:7:::
  lux:$1$AFvb7KAB$qg0IKBSFsGRDAO5XPaoYr/:17604:0:99999:7:::
  nginx:!!:17604::::::
  evobot:!$1$ORddLa4O$eScmBtXm8sIosft7XETeO1:17624:0:99999:7:::
  ```

- `usermod`命令锁定的账户，使用`usermod -U (username)`进行解锁:

  ```bash
  [root@evobot ~]# usermod -U evobot 
  [root@evobot ~]# tail /etc/shadow
  ntp:!!:17540::::::
  postfix:!!:17540::::::
  chrony:!!:17540::::::
  sshd:!!:17540::::::
  tcpdump:!!:17540::::::
  syslog:!!:17598::::::
  centos:!!:17602:0:99999:7:::
  lux:$1$AFvb7KAB$qg0IKBSFsGRDAO5XPaoYr/:17604:0:99999:7:::
  nginx:!!:17604::::::
  evobot:$1$ORddLa4O$eScmBtXm8sIosft7XETeO1:17624:0:99999:7:::

  ```


## passwd命令的其他用法

- `passwd`命令更改密码还可以使用`passwd --stdin (username)`为用户更改密码，这种方式更改密码只需要输入一次密码，并且密码是明文显示：

  ```bash
  [root@evobot ~]# passwd --stdin evobot 
  更改用户 evobot 的密码 。
  123456
  passwd：所有的身份验证令牌已经成功更新。
  ```

- 这种用法一般会在shell脚本里使用，常用的用法是`echo "112233" | passwd --stdin (username)`，这种方式不需要输入密码，`echo`后面的密码会在管道符`|`的作用下，传递给后面的`passwd`命令：

  ```bash
  [root@evobot ~]# echo "112233" | passwd --stdin evobot
  更改用户 evobot 的密码 。
  passwd：所有的身份验证令牌已经成功更新。

  ```

- 一行命令更改密码，也可以使用`echo -e`的用法，`echo -e`选项使得后面的内容里可以加上换行符`\n`或者制表符`\t`，所以更改密码可以用`echo -e "223344\n223344" | passwd (username)`的形式：

  ```bash
  [root@evobot ~]# echo -e "223344\n223344" | passwd evobot
  更改用户 evobot 的密码 。
  新的 密码：无效的密码： 密码少于 8 个字符
  重新输入新的 密码：passwd：所有的身份验证令牌已经成功更新。

  ```

# mkpasswd命令

- `mkdpasswd`是Linux中用来生成密码的命令行工具，默认这个命令是没有安装的，安装该命令，需要安装`expect`软件包：`yum install -y expect`;

- `mkpasswd`命令可以生成包含特殊符号，数字，字母大小写的随机字符串；

  ```bash
  [root@evobot ~]# mkpasswd 
  xEK0~wsn3
  ```

- 也可以指定生成的随机字符串的长度，使用`-l`选项，以及包含特殊符号个数，使用`-s`选项：

  ```bash
  [root@evobot ~]# mkpasswd -l 12	
  tio7lovMZd2}
  [root@evobot ~]# mkpasswd -l 12 -s 3
  hwg3R~2Qd[-h
  [root@evobot ~]# mkpasswd -l 12 -s 0
  y8zJSikw3qnt

  ```

---



