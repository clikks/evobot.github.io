---
title: Linux文件打包工具
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: b9ebaad1
date: 2018-04-15 23:10:03
image: https://s1.ax1x.com/2020/04/28/JINdfS.png
---



本文介绍了**zip**压缩工具的压缩解压缩操作，并且介绍Linux文件打包工具，由于压缩工具不支持对目录的压缩，所以实际使用中，常常会先进行文件或目录的打包，然后再进行压缩，本文将介绍LInux中的`tar`工具，能够实现对文件或目录的打包并压缩操作。

<!--more-->

---

# **zip**压缩工具

## 压缩操作

- 使用`zip`命令，需要先进行安装`zip`软件包，使用`yum install -y zip`进行安装；
- 对文件进行压缩，命令格式为`zip [压缩文件名] [源文件或目录]`:

```bash
[root@evobot compress]# zip conf.zip conf2.txt 
  adding: conf2.txt (deflated 75%)
  
[root@evobot compress]# ls -l
总用量 6448
-rw-r--r-- 1 root root 5195265 4月  14 23:26 conf2.txt
-rw-r--r-- 1 root root 1318260 4月  15 23:21 conf.zip
```

- zip压缩工具的`zip -r`选项能够实现对目录的压缩操作：

```bash
[root@evobot ~]# ls -l
总用量 4
drwxr-xr-x 2 lux root 4096 4月  15 23:20 source

[root@evobot ~]# zip -r source.zip source/
  adding: source/ (stored 0%)
  adding: source/go1.9.2.linux-amd64.zip (stored 0%)
  adding: source/test.txt (deflated 93%)
  adding: source/go.zip (stored 0%)
  
[root@evobot ~]# ls -l
总用量 492
drwxr-xr-x 2 lux  root   4096 4月  15 23:20 source
-rw-r--r-- 1 root root 492989 4月  15 23:23 source.zip
```

- `zip`压缩会自动保留压缩的源文件和目录。

## 解压缩操作

- 对**.zip**压缩文件进行解压缩操作，需要安装`unzip`软件包，使用`yum install -y unzip`安装；
- 解压文件直接使用`unzip [zip压缩文件]`即可，但是在目录下存在与压缩包内相同的文件，解压时会询问用户是否覆盖、替换或重命名操作：

```bash
[root@evobot ~]# unzip source.zip 
Archive:  source.zip
replace source/go1.9.2.linux-amd64.zip? [y]es, [n]o, [A]ll, [N]one, [r]ename: A	# 选择A则是全部替换文件
 extracting: source/go1.9.2.linux-amd64.zip  
  inflating: source/test.txt         
 extracting: source/go.zip           
```

- 如果想要在解压时不提示是否覆盖，可以使用`unzip -n`不覆盖原有文件选项，或者`unzip -o`直接覆盖原有文件选项：

```bash
[root@evobot ~]# unzip -n source.zip 
Archive:  source.zip
[root@evobot ~]# unzip -o source.zip 
Archive:  source.zip
 extracting: source/go1.9.2.linux-amd64.zip  
  inflating: source/test.txt         
 extracting: source/go.zip           
```

- `unzip`在解压时，可以指定解压的路径，使用`unzip [zip压缩文件] -d [解压目录]`，指定的目录不存在时，`unzip`会自动创建目录，这也表示在指定目录压缩时，无法对解压文件进行重命名操作：

```bash
[root@evobot ~]# unzip source.zip -d /tmp/
Archive:  source.zip
   creating: /tmp/source/
 extracting: /tmp/source/go1.9.2.linux-amd64.zip  
  inflating: /tmp/source/test.txt    
 extracting: /tmp/source/go.zip
 
[root@evobot ~]# unzip source.zip -d /tmp/newdir
Archive:  source.zip
   creating: /tmp/newdir/source/
 extracting: /tmp/newdir/source/go1.9.2.linux-amd64.zip 
  inflating: /tmp/newdir/source/test.txt  
 extracting: /tmp/newdir/source/go.zip  
```

- `.zip`压缩包无法查看压缩包内的文件内容，但是可以查看压缩包内的文件列表，使用`unzip -l [zip压缩包]`命令查看：

```bash
[root@evobot ~]# unzip -l source.zip 
Archive:  source.zip
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  04-15-2018 23:20   source/
   245760  03-12-2018 17:41   source/go1.9.2.linux-amd64.zip
     9051  03-27-2018 23:40   source/test.txt
   245956  04-15-2018 23:20   source/go.zip
---------                     -------
   500767                     4 files
```

---

# **tar**打包工具

- 打包是指将一大堆文件或目录变成一个总的文件，利用**tar**，可以为某一特定文件创建档案（备份文件），也可以在档案中改变文件，或者向档案中加入新的文件，也可以把一大堆的文件和目录全部打包成一个文件，这对于备份文件或将几个文件组合成为一个文件以便于网络传输是非常有用的
- Linux中很多压缩程序只能针对一个文件进行压缩，当想要压缩一大堆文件时，需要先将这一大堆文件先打成一个包（tar命令），然后再用压缩程序进行压缩（gzip，bzip2命令）。

## 打包操作

- 对目录进行打包，使用`tar -cvf [包名.tar] [目录]`，其中选项`-c`表示创建，`-v`表示显示打包过程，`-f`表示指定包名：

```bash
[root@evobot ~]# tar -cvf source.tar source
source/
source/go1.9.2.linux-amd64.zip
source/test.txt
source/go.zip

[root@evobot ~]# ls -l
总用量 996
drwxr-xr-x 2 lux  root   4096 4月  15 23:37 source
-rw-r--r-- 1 root root 512000 4月  15 23:53 source.tar

# 不使用-v选项
[root@evobot ~]# tar -cf source2.tar source
[root@evobot ~]# ls -l
总用量 1012
drwxr-xr-x 2 lux  root   4096 4月  15 23:37 source
-rw-r--r-- 1 root root 512000 4月  15 23:54 source2.tar
-rw-r--r-- 1 root root 512000 4月  15 23:53 source.tar
```

- 需要注意，如果打包时存在同名的包，则会不提示直接覆盖已存在的包；
- `tar`打包可以实现对目录、文件或者目录与文件一起的打包操作，只需要在打包时指定多个文件或目录即可：

```bash
[root@evobot log]# tar -cvf tmplog.tar anaconda/ dmesg lastlog maillog secure 
anaconda/
anaconda/packaging.log
anaconda/storage.log
anaconda/anaconda.log
anaconda/ks-script-KKMRDG.log
anaconda/ks-script-P2r9iH.log
anaconda/program.log
anaconda/ks-script-ixRdPZ.log
anaconda/journal.log
anaconda/syslog
anaconda/ifcfg.log
dmesg
lastlog
maillog
secure
[root@evobot log]# ls -l tmplog.tar 
-rw-r--r-- 1 root root 19834880 4月  16 00:02 tmplog.tar
```

- 在打包时，如果希望忽略部分文件，不对其进行打包，则使用`--exclude`选项，忽略多个文件时，需要添加多个`--exclude`选项，每个选项后跟一个文件名，另外忽略的文件也可以使用通配符匹配来忽略一类文件，具体用法如下：

```bash
[root@evobot log]# tar -cvf tmplog2.tar  --exclude anaconda/anaconda.log --exclude audit/audit.log.1 anaconda/ audit/
anaconda/
anaconda/packaging.log
anaconda/storage.log
anaconda/ks-script-KKMRDG.log
anaconda/ks-script-P2r9iH.log
anaconda/program.log
anaconda/ks-script-ixRdPZ.log
anaconda/journal.log
anaconda/syslog
anaconda/ifcfg.log
audit/
audit/audit.log
audit/audit.log.3
audit/audit.log.2
audit/audit.log.4

# 使用通配符忽略文件
[root@evobot log]# tar -cvf tmplog.tar --exclude anaconda/"*.log" --exclude audit/"audit.log.*" anaconda/ audit/
anaconda/
anaconda/syslog
audit/
audit/audit.log
```



## 解包操作

- 解包操作使用`tar -xvf [tar包]`命令，其中`-x`选项的作用是解包，解包时会直接覆盖原有文件：

```bash
[root@evobot ~]# tar -xvf source.tar 
source/
source/go1.9.2.linux-amd64.zip
source/test.txt
source/go.zip
```

- 查看tar包内的文件列表，使用`tar -tf [tar包]`命令：

```bash
[root@evobot log]# tar -tf tmplog.tar 
anaconda/
anaconda/packaging.log
anaconda/storage.log
anaconda/anaconda.log
anaconda/ks-script-KKMRDG.log
anaconda/ks-script-P2r9iH.log
anaconda/program.log
anaconda/ks-script-ixRdPZ.log
anaconda/journal.log
anaconda/syslog
anaconda/ifcfg.log
dmesg
lastlog
maillog
secure
```

- 解压时可以指定解压的路径，使用`-C`选项：

```bash
[root@evobot ~]# tar -xvf source.tar -C /tmp/
source/
source/go1.9.2.linux-amd64.zip
source/test.txt
[root@evobot ~]# ls -lhd /tmp/source/
drwxr-xr-x 2 lux root 4.0K 4月  16 00:20 /tmp/source/
```



## 打包并压缩

- `tar`在打包时，支持压缩操作，并且支持`.gz`、`.bz2`、`.xz`压缩格式，命令如下表：

<style>
table th:first-of-type {
    width: 180px;
}
</style>


|   压缩格式    |                   压缩命令                   |
| :-------: | :--------------------------------------: |
| **gzip**  | `tar -zcvf [filename.tar.gz] [file1] [file2] ...` |
| **bzip2** | `tar -jcvf [filename.tar.bz2] [file1] [file2] ...` |
|  **xz**   | `tar -Jcvf [filename.tar.xz][file1] [file2] ...` |

```bash
[root@evobot ~]# tar -zcvf source.tar.gz source/
source/
source/go1.9.2.linux-amd64.zip
source/test.txt
[root@evobot ~]# ls -lh source.tar.gz 
-rw-r--r-- 1 root root 241K 4月  16 00:26 source.tar.gz
[root@evobot ~]# file source.tar.gz 
source.tar.gz: gzip compressed data, from Unix, last modified: Mon Apr 16 00:26:05 2018

[root@evobot ~]# tar -jcvf source.tar.bz2 source
source/
source/go1.9.2.linux-amd64.zip
source/test.txt
[root@evobot ~]# ls -lh source.tar.bz2 
-rw-r--r-- 1 root root 244K 4月  16 00:27 source.tar.bz2
[root@evobot ~]# file source.tar.bz2 
source.tar.bz2: bzip2 compressed data, block size = 900k

[root@evobot ~]# tar -Jcvf source.tar.xz source
source/
source/go1.9.2.linux-amd64.zip
source/test.txt
[root@evobot ~]# ls -lh source.tar.xz 
-rw-r--r-- 1 root root 242K 4月  16 00:28 source.tar.xz
[root@evobot ~]# file source.tar.xz 
source.tar.xz: XZ compressed data

```

- 对压缩的tar包进行解压缩，只需要将压缩命令的`-c`选项替换为`-x`选项即可：

|   压缩格式    |              解压命令              |
| :-------: | :----------------------------: |
| **gzip**  | `tar -zxvf [filename.tar.gz]`  |
| **bzip2** | `tar -jxvf [filename.tar.bz2]` |
|  **xz**   | `tar -Jxvf [filename.tar.xz]`  |

```bash
[root@evobot ~]# tar -zxvf source.tar.gz 
source/
source/go1.9.2.linux-amd64.zip
source/test.txt

[root@evobot ~]# tar -jxvf source.tar.bz2 
source/
source/go1.9.2.linux-amd64.zip
source/test.txt

[root@evobot ~]# tar -Jxvf source.tar.xz 
source/
source/go1.9.2.linux-amd64.zip
source/test.txt
```

- 对压缩的tar包，查看包内的文件列表，依然使用`tar -tf [filename]`命令查看：

```bash
[root@evobot ~]# tar -tf source.tar.gz 
source/
source/go1.9.2.linux-amd64.zip
source/test.txt
```

---

