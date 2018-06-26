---
title: NFS服务介绍及安装配置
author: Evobot
categories: NFS
tags:
  - Linux
  - Centos
  - NFS
abbrlink: f352601c
date: 2018-06-21 23:26:40
image:
---



本文主要对NFS服务进行介绍，以及如何安装NFS服务和NFS的配置选项。

<!--more-->

---

# NFS介绍

- NFS是Network File System的缩写，分为2,3,4,三个版本，目前最新为4.1版本；

- NFS数据传输基于RPC协议，RPC是Remote Procedure Call的简写；

- NFS主要的应用场景为：A、B、C三台机器上需要保证被访问到的文件是一样的，A共享数据出来，B和C分别去挂载A共享的数据目录，从而B和C访问到的数据与A一致，架构如下图：

  ![NFS架构](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/NFS.png)

  原理图如下：

  ![NFS-theory](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/NFS-theory.png)

  > NFS服务本身不会监听端口，而是由rpcbind服务监听111端口，在Centos6以前，rpcbind被称为portmap。

# NFS服务安装与配置

## NFS安装

- 搭建NFS服务，需要安装`nfs-utils`和`rpcbind`这两个软件包，使用yum安装即可，同时在需要访问NFS服务的客户端上，同样也需要安装这两个软件包：

  ```bash
  yum install -y nfs-utils rpcbind
  ```

## 配置NFS服务端

- 在服务端，编辑`/etc/exports`文件，写入需要被共享访问的目录路径和允许访问的ip地址或ip段：

  ```bash
  [root@localhost ~]# vi /etc/exports
  # 写入如下内容
  /home/nfsdir 192.168.67.0/24(rw,sync,all_squash,anonuid=1000,anongid=1000)     
  ```

- 然后创建要共享的目录，并设置为777权限：

  ```bash
  [root@localhost ~]# mkdir /home/nfsdir
  [root@localhost ~]# chmod 777 nfsdir/
  ```

- 查看rpcbind服务是否启动，如果没有启动，使用`systemctl start rpcbind`启动服务：

  ```bash
  [root@localhost ~]# netstat -tlnp
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      868/sshd            
  tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1320/master         
        
  [root@localhost ~]# systemctl start rpcbind

  [root@localhost ~]# netstat -tlnp
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
  tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      3389/rpcbind        
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      868/sshd            
  tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1320/master         
  tcp6       0      0 :::111                  :::*                    LISTEN      3389/rpcbind        
  tcp6       0      0 :::22                   :::*                    LISTEN      868/sshd            
  tcp6       0      0 ::1:25                  :::*                    LISTEN      1320/master         

  [root@localhost ~]# ps aux |grep rpc
  rpc        3389  0.0  0.2  65008  1044 ?        Ss   00:10   0:00 /sbin/rpcbind -w
  root       3394  0.0  0.2 112724   972 pts/0    S+   00:10   0:00 grep --color=auto rpc

  ```

- 然后启动NFS服务：

  ```bash
  systemctl start nfs
  ```

  ```bash
  [root@localhost ~]# ps aux | grep nfs
  root       3436  0.0  0.0      0     0 ?        S<   00:13   0:00 [nfsd4_callbacks]
  root       3442  0.0  0.0      0     0 ?        S    00:13   0:00 [nfsd]
  root       3443  0.0  0.0      0     0 ?        S    00:13   0:00 [nfsd]
  root       3444  0.0  0.0      0     0 ?        S    00:13   0:00 [nfsd]
  root       3445  0.0  0.0      0     0 ?        S    00:13   0:00 [nfsd]
  root       3446  0.0  0.0      0     0 ?        S    00:13   0:00 [nfsd]
  root       3447  0.0  0.0      0     0 ?        S    00:13   0:00 [nfsd]
  root       3448  0.0  0.0      0     0 ?        S    00:13   0:00 [nfsd]
  root       3449  0.0  0.0      0     0 ?        S    00:13   0:00 [nfsd]

  #启动nfs服务后，会启动rpc关联服务
  [root@localhost ~]# ps aux | grep rpc
  rpc        3381  0.0  0.2  64948  1420 ?        Ss   00:13   0:00 /sbin/rpcbind -w
  rpcuser    3408  0.0  0.3  42364  1760 ?        Ss   00:13   0:00 /usr/sbin/rpc.statd
  root       3409  0.0  0.0      0     0 ?        S<   00:13   0:00 [rpciod]
  root       3412  0.0  0.0  19312   400 ?        Ss   00:13   0:00 /usr/sbin/rpc.idmapd
  root       3416  0.0  0.1  42548   940 ?        Ss   00:13   0:00 /usr/sbin/rpc.mountd
  root       3457  0.0  0.2 112664   968 pts/0    R+   00:15   0:00 grep --color=auto rpc
  ```

- 如果需要让NFS服务开机自启动，那么需要使用systemd进行设置：

  ```bash
  systemctl enable rpcbind
  systemctl enable nfs
  ```

## NFS配置选项

- 在`/etc/exports`文件中，我们在IP地址段后面还写了一些配置选项，具体的选项含义如下：

<style>
table th:first-of-type {
    width: 120px;
    text-align: center;
}
</style>

|        配置项        | 作用                                  |
| :---------------: | :---------------------------------- |
|       `rw`        | 读写                                  |
|       `ro`        | 只读                                  |
|      `sync`       | 同步模式，内存数据实时写入磁盘                     |
|      `async`      | 非同步模式，能够保证磁盘效率，但断电时可能会丢失数据。         |
| `no_root_squash`  | 客户端挂载NFS共享目录后，root用户不受约束，权限很大       |
|   `root_squash`   | 与上面的选项相对应，客户端上root用户收到约束，被限制为某个普通用户 |
|   `all_squash`    | 客户端上所有的用户在使用NFS共享目录时都被限定为一个普通       |
| `anonuid/anongid` | 与上面三个选项搭配使用，定义被限定用户的uid和gid         |

## NFS配置客户端

- 在NFS客户端上，使用命令`showmount -e [srcip]`查看服务端共享的目录：

  ```bash
  [root@localhost ~]# showmount 192.168.67.128
  clnt_create: RPC: Port mapper failure - Unable to receive: errno 113 (No route to host)

  ```

  > 产生报错，可能是服务端的服务未运行，也可能是防火墙或selinux导致端口未被放行，虽然rpcbind监听111端口，但nfs之间通信的端口是随机的，所以需要在服务端和客户端使用命令`systemctl stop firewalld`关闭防火墙。

- 关闭防火墙之后重新执行`showmount`命令：

  ```bash
  [root@localhost ~]# showmount -e 192.168.67.128
  Export list for 192.168.67.128:
  /home/nfsdir 192.168.67.0/24
  ```

  可以看到，服务端共享的目录为`/home/nfsdir`。

- 然后可以将共享目录挂载到本地：

  ```bash
  mount -t nfs 192.168.67.128:/home/nfsdir /mnt
  ```

  > 挂载时需要使用`-t`指定文件系统，然后使用`ip:/path`的形式指定被挂载的远程目录。

- 查看挂载：

  ```bash
  [root@localhost ~]# df -h
  文件系统                     容量  已用  可用 已用% 挂载点
  /dev/sda3                     28G  1.2G   27G    5% /
  devtmpfs                     227M     0  227M    0% /dev
  tmpfs                        237M     0  237M    0% /dev/shm
  tmpfs                        237M  4.5M  232M    2% /run
  tmpfs                        237M     0  237M    0% /sys/fs/cgroup
  /dev/sda1                    197M  109M   88M   56% /boot
  tmpfs                         48M     0   48M    0% /run/user/0
  192.168.67.128:/home/nfsdir   28G  5.3G   23G   19% /mnt

  ```

## NFS使用

- 在客户端的挂载目录内，创建一个测试文件，并且查看其权限和属组、属主：

  ```bash
  [root@localhost mnt]# touch test

  [root@localhost mnt]# ll
  总用量 0
  -rw-r--r--. 1 1000 1000 0 6月  22 00:43 test

  ```

- 在服务端的共享目录中查看文件是否同步，以及权限、属组、属主信息：

  ```bash
  [root@localhost ~]# cd /home/nfsdir/

  [root@localhost nfsdir]# ll
  总用量 0
  -rw-r--r--. 1 mysql mysql 0 6月  22 00:43 test

  ```

- 客户端和服务端上查看文件时，会发现属组和属主不同，这是因为我们设置了`anonuid`和`anongid`为1000，所以在服务端和客户端上查看时，会显示为uid、gid为1000的本地用户：

  ```bash
  # 服务端查看uid为1000的用户，而客户端没有uid为1000的用户，所以会直接显示uid和gid为1000
  [root@localhost nfsdir]# cat /etc/passwd | grep 1000
  mysql:x:1000:1000::/home/mysql:/sbin/nologin

  ```

---

