---
title: Linux文件压缩工具
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
image: 'http://p5qynomrl.bkt.clouddn.com/1523810699086ylm0327m.png?imageslim'
abbrlink: 701184ad
date: 2018-04-14 22:17:14
---



Linux中常见的压缩文件有**.gz**、**.bz2**、**.xz**、**tar.gz**、**.tar.bz2**、**.tar.xz**、**.zip**等等，对于服务器来说，对文件进行压缩，能够节省带宽资源，所以在Linux命令行下，也有相对应的命令工具来对文件进行压缩和解压缩操作。

<!--more-->

---

# **gzip**压缩工具

## 压缩操作

- `gzip`压缩工具压缩后的文件后缀名为`.gz`，对一个文件进行压缩操作时，可以使用命令`gzip [/path/to/filename]`：

```bash
[root@evobot compress]# du -sh conf.txt 
5.0M	conf.txt
[root@evobot compress]# gzip conf.txt 
[root@evobot compress]# ls
conf.txt.gz
[root@evobot compress]# du -sh conf.txt.gz 
1.3M	conf.txt.gz
```

- `gzip`在压缩时可以调整压缩级别，一共有9个级别，默认为`6`级别，压缩率最高的时`9`，相应的压缩率越高，越耗费CPU资源：

```bash
[root@evobot compress]# gzip -9 conf.txt 
[root@evobot compress]# du -sh conf.txt.gz 
1.3M	conf.txt.gz
[root@evobot compress]# gzip -d conf.txt.gz 
[root@evobot compress]# gzip -1 conf.txt 
[root@evobot compress]# du -sh conf.txt.gz 
1.5M	conf.txt.gz
```

- 默认直接使用`gzip`压缩文件时，不会保留源文件，也不能指定压缩文件的名字，想要保留源文件并指定新文件名，可以使用`gzip -c`选项，命令用法为`gzip -c [filename] > [/path/to/new.gz]`:

```bash
[root@evobot compress]# gzip -c conf.txt > /tmp/conf.gz
[root@evobot compress]# ls /tmp/
compress  conf.gz
[root@evobot compress]# ls
conf.txt
```

> gzip不可以对目录进行压缩。

## 解压操作

- 解压操作则使用`gzip -d`选项：

```bash
[root@evobot compress]# gzip -d conf.txt.gz 
[root@evobot compress]# ls
conf.txt
[root@evobot compress]# du -sh conf.txt 
5.0M	conf.txt
```

- `.gz`压缩包也可以使用`gunzip`命令进行解压缩：

```bash
[root@evobot compress]# gunzip conf.txt.gz 
[root@evobot compress]# ls -lh conf.txt 
-rw-r--r-- 1 root root 5.0M 4月  14 22:31 conf.txt
```

- `file`命令可以查看文件的格式，而使用命令`zcat`可以查看压缩包里的文档内容并在屏幕输出文件内容：

```bash
[root@evobot compress]# file conf.txt.gz 
conf.txt.gz: gzip compressed data, was "conf.txt", from Unix, last modified: Sat Apr 14 22:31:15 2018
```

- 与压缩相同，在解压缩的时候，使用`gzip -d -c`选项同样也可以保留压缩包，并将解压出的文件重命名:

```bash
[root@evobot compress]# gzip -d -c /tmp/conf.gz > conf2.txt
[root@evobot compress]# ls
conf2.txt  conf.txt
[root@evobot compress]# ls /tmp/
compress  conf.gz
```

---

# **bzip2**压缩工具

- `bzip2`相比`gzip`压缩率更高，压缩的后缀名是`bz2`，同样不支持压缩目录，基本操作基本与`gzip`加压缩相同。


- 使用`bzip2 [filename]`进行压缩操作，选项`-d`进行解压操作，压缩解压缩时也可以使用`-c`选项，作用与`gzip`的命令选项作用相同，也可以使用`-[num]`指定压缩级别：

```bash
[root@evobot compress]# bzip2 conf2.txt 
[root@evobot compress]# ls
conf2.txt.bz2  conf.txt

[root@evobot compress]# bzip2 -d conf2.txt.bz2 
[root@evobot compress]# ls
conf2.txt  conf.txt

[root@evobot compress]# bzip2 -c conf.txt > /tmp/bzip2_conf.bz2
[root@evobot compress]# ls
conf2.txt  conf.txt
[root@evobot compress]# ls -lh /tmp/bzip2_conf.bz2 
-rw-r--r-- 1 root root 528K 4月  14 23:06 /tmp/bzip2_conf.bz2

[root@evobot compress]# bzip2 -d -c /tmp/bzip2_conf.bz2 > conf3.txt
[root@evobot compress]# ls
conf2.txt  conf3.txt  conf.txt

[root@evobot compress]# bzip2 -9 conf.txt 
[root@evobot compress]# ls -lh conf
conf2.txt     conf3.txt     conf.txt.bz2  
[root@evobot compress]# ls -lh conf.txt.bz2 
-rw-r--r-- 1 root root 528K 4月  14 22:31 conf.txt.bz2

[root@evobot compress]# bzip2 -1 conf2.txt 
[root@evobot compress]# ls -lh conf2.txt.bz2 
-rw-r--r-- 1 root root 1.3M 4月  14 22:55 conf2.txt.bz2

```

- 同样的，`bz2`压缩包也有一个`bzcat`命令，可以用来查看压缩包内的文件内容。

---

# **xz**压缩工具

- `xz`压缩工具的压缩率相比其他两个更高，后缀名为`xz`，同样也不能够压缩目录，并且基本的命令操作也与`gzip`、`bzip2`相同。

```bash
[root@evobot compress]# xz conf.txt 
[root@evobot compress]# ls -lh
总用量 68K
-rw-r--r-- 1 root root 61K 4月  14 22:31 conf.txt.xz

[root@evobot compress]# xz -d conf.txt.xz 
[root@evobot compress]# ls -lh
总用量 5.0M
-rw-r--r-- 1 root root 5.0M 4月  14 22:31 conf.txt

[root@evobot compress]# xz -c conf.txt > /tmp/xz_conf.xz
[root@evobot compress]# ls -lh /tmp/
总用量 76K
drwxrwxr-x 2 lux  lux  4.0K 4月  14 23:25 compress
-rw-r--r-- 1 root root  61K 4月  14 23:26 xz_conf.xz

[root@evobot compress]# xz -d -c /tmp/xz_conf.xz > conf2.txt
[root@evobot compress]# ls -lh
总用量 10M
-rw-r--r-- 1 root root 5.0M 4月  14 23:26 conf2.txt
-rw-r--r-- 1 root root 5.0M 4月  14 22:31 conf.txt
[root@evobot compress]# ls -lh /tmp/
总用量 76K
drwxrwxr-x 2 lux  lux  4.0K 4月  14 23:26 compress
-rw-r--r-- 1 root root  61K 4月  14 23:26 xz_conf.xz

[root@evobot compress]# xz -9 conf.txt 
[root@evobot compress]# ls -lh
总用量 5.1M
-rw-r--r-- 1 root root 5.0M 4月  14 23:26 conf2.txt
-rw-r--r-- 1 root root  61K 4月  14 22:31 conf.txt.xz

[root@evobot compress]# xz -1 conf2.txt
[root@evobot compress]# ls -lh
总用量 140K
-rw-r--r-- 1 root root 67K 4月  14 23:26 conf2.txt.xz
-rw-r--r-- 1 root root 61K 4月  14 22:31 conf.txt.xz
```

- 解压`.xz`压缩包还可以使用`unxz [filename]`，查看压缩包内的文件，则使用`xzcat`命令。

```bash
[root@evobot compress]# unxz conf2.txt.xz 
[root@evobot compress]# ls -lh
总用量 5.1M
-rw-r--r-- 1 root root 5.0M 4月  14 23:26 conf2.txt
-rw-r--r-- 1 root root  61K 4月  14 22:31 conf.txt.xz
```

---

