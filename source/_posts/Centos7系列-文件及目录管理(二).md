---
title: 'Centos7系列:文件及目录管理相关命令(二)'
author: Evobot
abbrlink: fd76e245
date: 2018-03-26 21:50:06
categories: Centos7
tags: [Linux, Centos]
image: http://p5qynomrl.bkt.clouddn.com/1522075826408alwrya2w.png?imageslim
---
本文主要介绍Linux相对和绝对路径的概念，以及cd命令、mkdir、rmdir命令和rm几个操作文件及目录的命令。文章封面的`rm -rf /*`命令会导致系统崩溃，请勿尝试~	

<!-- more -->

# 相对及绝对路径

## 绝对路径

- 由于Linux的目录结构是树型结构，所以从根目录开始的路径，就称之为绝对路径，比如网卡配置文件：

  ```bash
  [lux@evobot ~]$ ls /etc/sysconfig/network-scripts/ifcfg-eth0 
  /etc/sysconfig/network-scripts/ifcfg-eth0
  ```

- 绝对路径能够保证我们不论在什么位置，都能找到自己想要的文件或目录。

## pwd命令

- 当我们想知道自己当前所在目录的绝对路径时，可以执行`pwd`命令得到当前路径的绝对路径:

  ```bash
  [lux@evobot ~]$ pwd
  /home/lux
  ```

## 相对路径

- 相对路径的意思是指相对于当前所在目录的路径，比如在家目录，需要查看`.ssh`目录时，并不需要输入从根目录开始的绝对路径，例如：

  ```bash
  [root@evobot ~]# ls .ssh/authorized_keys 
  .ssh/authorized_keys
  ```

- 可以从上面看到，`.ssh`相对当前目录家目录，可以直接查看而不需要输入绝对路径查找文件。

# cd命令

- `cd`命令用于进入到目录，如进入根目录`cd /`，另外`cd`命令还有一些小技巧，如切换到上次的目录`cd -`，切换到上层目录`cd ..`;
- 直接输入`cd`命令，则会进入到当前用户的家目录下，同样的`cd ~`具有相同的功能。

# 创建和删除目录

## mkdir创建目录

- `mkdir`命令用于创建目录，可以使用绝对路径和相对路径的方式：

  ```bash
  [root@evobot ~]# mkdir /tmp/evobot1
  [root@evobot ~]# mkdir evobot2
  [root@evobot ~]# ls /tmp/
  evobot1
  [root@evobot ~]# ls 
  evobot2
  ```

- 创建多级目录使用选项`mkdir -p`：

  ```bash
  [root@evobot ~]# mkdir -p evobot3/dir1
  [root@evobot ~]# ls
  evobot2  evobot3
  [root@evobot ~]# ls evobot3/
  dir1
  ```

- `mkdir -v`选项可以可视化创建目录，即显示创建目录的过程:

  ```bash
  [root@evobot ~]# mkdir -pv evobot3/dir2/dir3/dir4
  mkdir: 已创建目录 "evobot3/dir2"
  mkdir: 已创建目录 "evobot3/dir2/dir3"
  mkdir: 已创建目录 "evobot3/dir2/dir3/dir4"
  ```

## 删除目录

- 删除目录可以使用`rmdir`命令，但是删除的前提时目录为空，即目录存在目录和文件时，都无法删除：

  ```bash
  [root@evobot ~]# rmdir evobot3/dir2
  rmdir: 删除 "evobot3/dir2" 失败: 目录非空
  ```

- `rmdir`同样有`-p`选项，可以级联删除，但是容易出现操作失误，因为`-p`操作会将目录下空目录均删除。

## rm删除命令

### 删除文件

- `rm`命令也是删除的命令，而且可以删除文件和目录：

  ```bash
  [root@evobot ~]# touch evobot3/dir2/dir3/dir4/test1
  [[root@evobot ~]# tree  evobot3/
  evobot3/
  |-- dir1
  `-- dir2
      `-- dir3
          `-- dir4
              `-- test1

  4 directories, 1 file
  [root@evobot ~]# rm evobot3/dir2/dir3/dir4/test1 
  rm：是否删除普通空文件 "evobot3/dir2/dir3/dir4/test1"？y
  ```

- `rm`命令在操作时会寻味是否删除，输入`y`即可同意删除文件；

- `touch`命令用来创建一个空文件，使用方法为`touch (文件名)`;

- 如果不想在使用`rm`删除文件时提示是否删除，可以使用`rm -f`选项，强制删除文件:

  ```bash
  [root@evobot ~]# !touch	# 重复执行最后一次执行过的touch命令
  touch evobot3/dir2/dir3/dir4/test1
  [root@evobot ~]# rm -f evobot3/dir2/dir3/dir4/test1 
  ```

### 删除目录

- `rm`删除目录需要使用`-r`选项，同样在未使用`-f`选项时，会交互式询问是否删除：

  ```bash
  [root@evobot ~]# rm -r evobot3/dir2/dir3/
  rm：是否进入目录"evobot3/dir2/dir3/"? y
  rm：是否删除目录 "evobot3/dir2/dir3/dir4"？y
  rm：是否删除目录 "evobot3/dir2/dir3/"？y
  [root@evobot ~]# rm -rf evobot3/dir
  dir1/ dir2/ 
  [root@evobot ~]# rm -rf evobot3/dir2/
  ```

- 当使用`-f`选项时，即使文件或目录不存在，也不会提示出错。

---

