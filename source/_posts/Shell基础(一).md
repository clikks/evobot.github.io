---
title: Shell基础(一)
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: e6d37612
date: 2018-04-20 23:05:06
image:
---



本文将介绍shell的功能和一些操作用法，介绍shell的命令历史，命令补全以及命令别名，并使用通配符提高shell操作效率，另外还将介绍shell中如何将输入和输入重定向，如将本应输出到屏幕的命令输出到文件中。

<!--more-->

---

# Shell基础(一)

## Shell介绍

- shell是一个命令解释器，提供用户和机器之间的交互；
- 并且shell支持特定的语法，如逻辑判断、循环等等；
- 每个用户可以有自己特定的shell，在Centos7中，默认shell为bash，除了bash以外，还有zsh，ksh等等，需要自己安装体验。

## 命令历史

### 查看命令历史

- 在shell中可以使用上下方向键找回之前执行过的命令，实际上这些命令是存储在`~/.bash_history`文件中；
- 使用命令`history`命令可以查看到历史执行过的命令，默认情况下，在`.bash_history`文件中会存储1000条命令；
- 历史命令的条数是在系统环境变量`$HISTSIZE`中定义的，使用`history -c`可以清空内存中的历史命令，但不能清空文件中记录的历史命令；
- 用户执行过的命令，在退出登录前都是存储在内存中，退出登录后才会写入到`.bash_history`文件中去；

### 配置命令历史

- 修改历史命令记录的大小，可以修改`/etc/profile`文件：

```bash
HOSTNAME=`/usr/bin/hostname 2>/dev/null`
HISTSIZE=1000	# 修改这里的数字即可改变历史命令记录的条数
if [ "$HISTCONTROL" = "ignorespace" ] ; then
    export HISTCONTROL=ignoreboth
else
    export HISTCONTROL=ignoredups
fi
```

- 修改完`/etc/profile`之后，在下次登录时生效，如果想要立即生效，可以执行`source /etc/profile`命令；
- 默认配置记录的命令历史并没有命令执行的时间，可以定义`$HISTTIMEFORMAT`变量定义命令历史格式，如修改为年/月/日 时:分:秒：

```bash
[root@localhost etc]# echo $HISTTIMEFORMAT

[root@localhost etc]# HISTTIMEFORMAT="%Y/%m/%d %H:%M:%S"
[root@localhost etc]# echo $HISTTIMEFORMAT
%Y/%m/%d %H:%M:%S

[root@localhost etc]# history 
    1  2018/04/20 22:54:45ip addr
    2  2018/04/20 22:54:45dhclient
    3  2018/04/20 22:54:45ip addr
    4  2018/04/20 22:54:45ping www.baidu.com
    5  2018/04/20 22:54:45init 6
    6  2018/04/20 22:54:45ip a
    7  2018/04/20 22:54:45init 0
    8  2018/04/20 22:54:45fdisk -l
    9  2018/04/20 22:54:45fdisk /dev/sdb
```

- 配置命令历史格式为默认格式，只需要将`HISTTIMEFORMAT`变量写入`/etc/profile`中：

```bash
HOSTNAME=`/usr/bin/hostname 2>/dev/null`
HISTSIZE=1000
HISTTIMEFORMAT="%Y/%m/%d %H:%M:%S"	# 增加变量
if [ "$HISTCONTROL" = "ignorespace" ] ; then
    export HISTCONTROL=ignoreboth
else
    export HISTCONTROL=ignoredups
fi
```

- 记录命令历史的`.bash_history`文件，可以追溯之前所以执行过的命令，所以可以将其设置为永久保存，给`.bash_history`增加隐藏权限`a`，执行命令`chattr +a ~/.bash_history`增加权限；
- 在shell中使用`!!`可以重新执行上一条命令，重复运行指定的命令，使用`!n`其中`n`是命令历史的编号，`!word`其中word是之前执行过的命令的开头单词，这样可以重复执行匹配到的最近一条同样word开头的命令：

```bash
[root@localhost etc]# !!
pwd
/etc

[root@localhost etc]# history | head
    1  2018/04/20 22:54:45ip addr
    2  2018/04/20 22:54:45dhclient
    3  2018/04/20 22:54:45ip addr
    4  2018/04/20 22:54:45ping www.baidu.com
    5  2018/04/20 22:54:45init 6
    6  2018/04/20 22:54:45ip a
    7  2018/04/20 22:54:45init 0
    8  2018/04/20 22:54:45fdisk -l
    9  2018/04/20 22:54:45fdisk /dev/sdb
   10  2018/04/20 22:54:45fg
   
[root@localhost etc]# !3
ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:4e:b0:a6 brd ff:ff:ff:ff:ff:ff
    inet 192.168.199.224/24 brd 192.168.199.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::7056:ab3e:1b19:7d4a/64 scope link 
       valid_lft forever preferred_lft forever
       
[root@localhost etc]# !echo
echo "hello world"
hello world
```

## 命令补全

- 使用tab键可以在shell中补全命令或文件路径，当补全的命令或文件存在2个或多个时，需要敲两次tab，只有一个时，敲一次tab即可补全；
- 增强bash命令补全功能，可以安装`bash-completion`软件包，安装完成后重启，即可在bash内补全命令的参数；

## 命令别名

- 命令别名使用`alias`命令，用户自己的别名设置，在`~/.bashrc`文件内；
- 在`/etc/profile.d/`目录下，存在多个shell脚本文件，这是系统定义的命令别名文件；
- 使用`unalias [别名]`就可以取消设置的命令别名。

## 通配符

- 通配符是指能够指示一类文件的符号，使用通配符匹配文件，能够更快对需要的同一类文件进行处理；
- 常用的通配符如下表：

|              通配符              |       命令形式       |      作用      |
| :---------------------------: | :--------------: | :----------: |
|              `*`              |    `ls *.txt`    |  匹配0个或多个字符   |
|              `?`              |    `ls ?.txt`    |   匹配一个任意字符   |
| `[0-9]` `[a-z]` `[0-9a-zA-Z]` |  `ls [0-3].txt`  | 匹配指定范围内的一个字符 |
|            `{1,2}`            | `ls {1.2.3.a}.txt` |  匹配括号内的一个字符  |

## 输入输出重定向

- 输入输出重定向，可以将命令的输入和输出指向别的命令或文件；
- 常见的重定向符号如下：

|      重定向符号      |                 命令形式                  |             作用             |
| :-------------: | :-----------------------------------: | :------------------------: |
|       `>`       |          `cat 1.txt > 2.txt`          | 将符号左边命令的**正确输出**覆盖写入到符号右边  |
|      `>>`       |         `cat 1.txt >> 2.txt`          | 将符号左边命令的**正确输出**追加到符号右边文件  |
|      `2>`       |          `ls aaa.txt 2> err`          | 将符号左边命令的**错误执行**信息，覆盖写入到符号 |
|      `2>>`      |       `ls test.txt 2>> err.txt`       | 将符号左边命令的**错误执行**信息，追加写入到符号 |
|      `&>`       |    `ls [12].txt aa.txt &> err.txt`    | 将**正确和错误**的执行信息，都覆盖输出到符号右边 |
|      `&>>`      |   `ls [12].txt aa.txt &>> err.txt`    |  将**正确和错误**的执行信息，都输出到符号右边  |
| `cmd > x 2> y` |   `ls 1.txt a.txt > true 2> false`    |    将命令执行正确的输出到x，错误的输出到y    |
| `cmd > X 2>&1`  | `ls /etc/hosts /etc/xzfa > true 2>&1` |  将命令执行的结果不论正确与否，都输出到X文件中   |
|       `<`       |            `wc -l < 1.txt`            | 输入重定向。将符号右边的内容输入到左边的**命令** |

---

