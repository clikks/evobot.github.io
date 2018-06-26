---
title: xftp和pure-ftpd的配置与使用
author: Evobot
categories: FTP
tags:
  - xftp
  - pure-ftpd
  - Linux
abbrlink: 75c15ce8
date: 2018-06-24 21:23:34
image:
---



本文介绍了xshell中的xftp软件的使用，即sftp工具，另外则介绍如何使用pure-ftpd搭建FTP服务。

<!--more-->

---

# xftp的使用

- xftp是与xshell搭配使用的sftp工具，在netsarang官网下载xftp软件并安装，xftp只有windows版本。
- 在xshell中，使用`ctrl+alt+f`即可打开当前登录机器的xftp窗口，左边为本地目录，右侧为远程机器的目录，直接拖拽或双击即可下载或上传需要的文件。

# pure-ftpd安装配置

## 安装配置

- pure-ftpd相比vsftpd更加轻量和简单，使用pure-ftpd需要安装epel软件源的`pure-ftpd`软件包;

- 然后打开配置文件`/etc/pure-ftpd.conf`，找到`pureftpd.pdb`，删掉注释保存退出，pureftpd.pdb是pure-ftpd的密码配置文件；

- 启动pure-ftpd服务，如果之前启动了vsftpd，需要先关闭vsftpd：

  ```bash
  systemctl start pure-ftpd
  ```

- 创建ftp共享目录和pure-ftp用户，然后将ftp目录的属组和属主改为新建的用户：

  ```bash
  [root@evobot ~]# mkdir /opt/ftp
  [root@evobot ~]# useradd -u 1010 pure-ftp
  [root@evobot ~]# chown -R pure-ftp.pure-ftp /opt/ftp/
  ```

- 使用`pure-pw useradd`命令创建用户，命令如下：

  ```bash
  pure-pw useradd ftp_usera -u pure-ftp -d /opt/ftp/
  ```

  `useradd`后面是指定ftp的虚拟用户名，`-u`选项指定虚拟用户映射到系统的用户名，`-d`指定虚拟用户的家目录，即ftp共享目录。执行命令后会要求为虚拟用户创建密码。

- 然后执行`pure-pw mkdb`是生成pure-ftpd识别的密码文件;

## 测试服务

- 在`/opt/ftp`目录下创建ftptest.txt测试文件，然后使用lftp或其他ftp客户端，登录pure-ftpd提供的ftp服务：

  ```bash
  [root@evobot ~]# touch /opt/ftp/ftptest.txt

  [root@evobot ~]# lftp ftp_usera@127.0.0.1
  口令: 
  lftp ftp_usera@127.0.0.1:~> ls      
  drwxr-xr-x    2 1010       pure-ftp         4096 Jun 25 00:28 .
  drwxr-xr-x    2 1010       pure-ftp         4096 Jun 25 00:28 ..
  -rw-r--r--    1 0          0                   0 Jun 25 00:28 ftptest.txt

  ```

  > 成功登录，但由于文件是root创建，所以更改ftptest.txt的属主和属组再重新登录

  ```bash
  [root@localhost ~]# chown pure-ftp.pure-ftp /opt/ftp/ftptest.txt 

  [root@localhost ~]# lftp ftp-usera@127.0.0.1
  口令: 
  lftp ftp-usera@127.0.0.1:~> ls      
  drwxr-xr-x    2 1003       pure-ftp           25 Jun 25 00:36 .
  drwxr-xr-x    2 1003       pure-ftp           25 Jun 25 00:36 ..
  -rw-r--r--    1 1003       pure-ftp            0 Jun 25 00:36 ftptest.txt

  ```

  > 可以看到，在ftp中，文件的属主显示的是uid。

---

