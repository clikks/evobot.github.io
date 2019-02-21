---
title: 自动化运维saltstack
author: Evobot
categories: 自动化运维
tags:
  - saltstack
abbrlink: '17498151'
date: 2018-09-18 21:22:48
image:
---

1. 自动化运维介绍
2. saltstack安装
3. 启动saltstack服务
4. saltstack配置认证

<!--more-->

---

# 自动化运维介绍

- 由于传统运维效率低、大多数工作由人工完成，并且工作繁琐、重复，容易出错，没有标准化流程，导致运维的脚本繁多，不能方便管理，而自动化运维就是用来解决上面的这些问题；
- 常见的自动化运维工具有以下几种：
  1. Puppet：基于ruby开发，c/s架构，支持多平台，可管理配置文件、用户、cron任务、软件包、系统服务等，分为免费的社区版和收费的企业版，企业版支持图形化配置。
  2. Saltstack：基于python开发，c/s架构，支持多平台，比Puppet轻量，在远程执行命令时非常快捷，配置和使用比puppet容易，能实现puppet几乎所有的功能；
  3. Ansible：更加简洁的自动化运维工具，不需要在客户端上安装agent，基于python开发，可以实现批量操作系统配置、批量程序部署、批量运行命令。

# saltstack安装

- saltstack可以使用salt-ssh远程执行，类似ansible；也支持c/s架构，这里主要介绍c/s架构的使用；

- 首先准备两台机器，一台为服务端，IP为192.168.49.128，hostname为vm1，一台为客户端，IP为192.168.49.129，hostname为vm2，使用`hostnamectl set-hostname [hostname]`设置机器的hostname；

- 两台机器都要安装saltstack的yum源，使用下面的命令安装yum源：

  ```bash
  yum install https://repo.saltstack.com/yum/redhat/salt-repo-latest-2.el7.noarch.rpm 
  ```

- 然后在服务端上安装`salt-master`和`salt-minion`软件包，而客户端只需要安装`salt-minion`软件包，

- 两台机器还需要设置hosts，内容如下：

  ```bash
  192.168.49.128 vm1
  192.168.49.129 vm2
  ```

# saltstack服务及配置

## 启动saltstack相关服务

- 在服务端上编辑配置文件`/etc/salt/minion`,找到`master: salt`配置，去掉注释，改为`master: vm1`；

- 在客户端上同样编辑配置文件，将`master: salt`改为`master: vm1`

- 然后使用下面的命令启动服务端上的saltstack服务：

  ```bash
  //先启动salt-minion服务
  systemctl start salt-minion
  //再启动salt-master服务
  systemctl start salt-master
  ```

- 如果启动报错提示信息如下，则需要更新python的psutil包，使用`pip install psutil --upgrade`进行更新：

  ```bash
  [root@vm1 ~]# systemctl status salt-master
  ● salt-master.service - The Salt Master Server
     Loaded: loaded (/usr/lib/systemd/system/salt-master.service; disabled; vendor preset: disabled)
     Active: failed (Result: exit-code) since 二 2018-09-18 22:45:55 CST; 3min 39s ago
       Docs: man:salt-master(1)
             file:///usr/share/doc/salt/html/contents.html
             https://docs.saltstack.com/en/latest/contents.html
    Process: 2399 ExecStart=/usr/bin/salt-master (code=exited, status=1/FAILURE)
   Main PID: 2399 (code=exited, status=1/FAILURE)
  
  9月 18 22:45:55 vm1 salt-master[2399]: import salt.config as config
  9月 18 22:45:55 vm1 salt-master[2399]: File "/usr/lib/python2.7/site-packages/s...e>
  9月 18 22:45:55 vm1 salt-master[2399]: import psutil
  9月 18 22:45:55 vm1 salt-master[2399]: File "/usr/lib64/python2.7/site-packages...e>
  9月 18 22:45:55 vm1 salt-master[2399]: import psutil._pslinux as _psplatform
  9月 18 22:45:55 vm1 salt-master[2399]: AttributeError: 'module' object has no a...x'
  9月 18 22:45:55 vm1 systemd[1]: salt-master.service: main process exited, code...URE
  9月 18 22:45:55 vm1 systemd[1]: Failed to start The Salt Master Server.
  9月 18 22:45:55 vm1 systemd[1]: Unit salt-master.service entered failed state.
  9月 18 22:45:55 vm1 systemd[1]: salt-master.service failed.
  Hint: Some lines were ellipsized, use -l to show in full.
  
  ```

- 在服务端上，可以查看saltstack运行的服务和端口：

  ```bash
  [root@vm1 ~]# ps aux |grep aslt
  root       3935  0.0  0.2 112724   972 pts/0    S+   22:59   0:00 grep --color=auto aslt
  [root@vm1 ~]# ps aux |grep salt
  root       2468  0.0  1.4 314060  6988 ?        Ss   22:52   0:00 /usr/bin/python /usr/bin/salt-minion
  root       2471  0.2  7.5 572724 36372 ?        Sl   22:52   0:01 /usr/bin/python /usr/bin/salt-minion
  root       2479  0.0  3.2 409020 15556 ?        S    22:52   0:00 /usr/bin/python /usr/bin/salt-minion
  root       2548  0.1  7.3 394052 35688 ?        Ss   22:52   0:00 /usr/bin/python /usr/bin/salt-master
  root       2553  0.0  4.2 311060 20700 ?        S    22:52   0:00 /usr/bin/python /usr/bin/salt-master
  root       2558  0.0  7.1 474936 34408 ?        Sl   22:52   0:00 /usr/bin/python /usr/bin/salt-master
  root       2559  0.0  7.1 393396 34632 ?        S    22:52   0:00 /usr/bin/python /usr/bin/salt-master
  root       2562  0.9 10.5 419492 51248 ?        S    22:52   0:03 /usr/bin/python /usr/bin/salt-master
  root       2563  0.0  7.2 394052 35312 ?        S    22:52   0:00 /usr/bin/python /usr/bin/salt-master
  root       2564  1.9  7.2 770908 35248 ?        Sl   22:52   0:07 /usr/bin/python /usr/bin/salt-master
  root       2565  0.5  9.9 492556 48304 ?        Sl   22:52   0:02 /usr/bin/python /usr/bin/salt-master
  root       2572  0.1  7.3 467784 35628 ?        Sl   22:52   0:00 /usr/bin/python /usr/bin/salt-master
  root       2573  0.5  9.9 492532 48268 ?        Sl   22:52   0:02 /usr/bin/python /usr/bin/salt-master
  root       2574  0.5 10.0 492532 48536 ?        Sl   22:52   0:02 /usr/bin/python /usr/bin/salt-master
  root       2576  0.5  9.9 492540 48284 ?        Sl   22:52   0:02 /usr/bin/python /usr/bin/salt-master
  root       2577  0.5  9.9 492536 48280 ?        Sl   22:52   0:02 /usr/bin/python /usr/bin/salt-master
  
  ```

  - 可以看到saltstack的服务实际上是由python运行的；

  ```bash
  [root@vm1 ~]# netstat -tlnp
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1272/sshd           
  tcp        0      0 0.0.0.0:4505            0.0.0.0:*               LISTEN      2558/python         
  tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1968/master         
  tcp        0      0 0.0.0.0:4506            0.0.0.0:*               LISTEN      2564/python         
  tcp6       0      0 :::22                   :::*                    LISTEN      1272/sshd           
  tcp6       0      0 ::1:25                  :::*                    LISTEN      1968/master         
  ```

  - 其中4505,4506就是saltstack服务端监听的端口，其中4505是用来发布消息的，是saltstack的`zeromq`消息队列依赖包使用的，4506则是master和minion之间用来通信的端口。

- 客户端上只需要启动`salt-minion`服务，服务启动后并不会监听任何端口：

  ```bash
  systemctl start salt-minion
  ```

  ```bash
  [root@vm2 ~]# ps aux |grep salt
  root       2422  0.0  4.4 313772 21396 ?        Ss   22:58   0:00 /usr/bin/python /usr/bin/salt-minion
  root       2425  0.3  8.8 567280 42648 ?        Sl   22:58   0:01 /usr/bin/python /usr/bin/salt-minion
  root       2433  0.0  4.1 404068 20152 ?        S    22:58   0:00 /usr/bin/python /usr/bin/salt-minion
  
  [root@vm2 ~]# netstat  -tlnp
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      991/sshd            
  tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1469/master         
  tcp6       0      0 :::22                   :::*                    LISTEN      991/sshd            
  tcp6       0      0 ::1:25                  :::*                    LISTEN      1469/master     
  ```

## 认证配置

- master端和minion端通信需要建立一个安全通道，传输过程需要加密，所以得配置认证，也是通过密钥对来进行加密和解密的；

- minion在第一次启动时，会在`/etc/salt/pki/minion/`下生成`minion.pem`和`minion.pub`两个文件，其中`.pub`为公钥，他会把公钥传输给master，但只有在进行认证的时候才会进行传输：

  ```bash
  [root@vm2 ~]# ls /etc/salt/pki/minion/
  minion.pem  minion.pub
  
  ```

- master第一次启动时，也会在`/etc/salt/pki/master`下生成密钥对，当master收到minion传过来的公钥后，通过salt-key工具接受这个公钥，一旦接受后，就会在`/etc/salt/pki/master/minion/`目录下存放接受的公钥，并更名为minion客户端主机名，同时客户端也会接受master传过去的公钥，并把它放在`/etc/salt/pki/minion/`目录下，命名为`minion_master.pub`：

  ```bash
  [root@vm1 ~]# ls /etc/salt/pki/master/
  master.pem  master.pub  minions  minions_autosign  minions_denied  minions_pre  minions_rejected
  
  ```

- 上面的认证过程，需要借助salt-key工具来实现，在服务端上执行`salt-key -a [client_hostname]`命令，可以认证制定的主机：

  ```bash
  [root@vm1 ~]# salt-key -a vm2
  The following keys are going to be accepted:
  Unaccepted Keys:
  vm2
  Proceed? [n/Y] y
  Key for minion vm2 accepted.
  
  ```

- 认证完成后，可以直接执行`salt-key`命令查看认证的秘钥：

  ```bash
  [root@vm1 ~]# salt-key 
  Accepted Keys:
  vm2
  Denied Keys:
  Unaccepted Keys:
  vm1
  Rejected Keys:
  
  ```

- 然后在服务端的目录内，可以看到客户端的公钥：

  ```bash
  [root@vm1 ~]# ls /etc/salt/pki/master/minions
  vm2
  
  ```

- 客户端同样会保存服务端的公钥：

  ```bash
  [root@vm2 ~]# ls /etc/salt/pki/minion/
  minion_master.pub  minion.pem  minion.pub
  
  ```

- `salt-key`的`-a`选项，只能认证指定的主机，而`-A`选项，则可以一次认证所有的客户端，关于`salt-key`命令，还有以下几个选项：

  - `-r`：跟主机名，拒绝指定主机；
  - `-R`：拒绝所有主机；
  - `-d`：跟主机名，删除主机认证；
  - `-D`：删除所有主机认证；
  - `-y`：省略交互，直接确认认证，相当于上面认证过程中不再需要按y。

  ```bash
  [root@vm1 ~]# salt-key -D
  The following keys are going to be deleted:
  Accepted Keys:
  vm2
  Unaccepted Keys:
  vm1
  Proceed? [N/y] y
  Key for minion vm1 deleted.
  Key for minion vm2 deleted.
  [root@vm1 ~]# salt-key -A
  The following keys are going to be accepted:
  Unaccepted Keys:
  vm1
  Proceed? [n/Y] y
  Key for minion vm1 accepted.
  [root@vm1 ~]# salt-key
  Accepted Keys:
  vm1
  Denied Keys:
  Unaccepted Keys:
  Rejected Keys:
  
  ```

  - 这里会发现删除所有认证之后再重新认证，会无法找到vm2，解决的办法是重启客户端的`salt-minion`服务，然后再服务端重新进行认证即可：

  ```bash
  [root@vm2 ~]# systemctl restart salt-minion
  [root@vm1 ~]# salt-key -A
  The following keys are going to be accepted:
  Unaccepted Keys:
  vm2
  Proceed? [n/Y] y
  Key for minion vm2 accepted.
  [root@vm1 ~]# salt-key 
  Accepted Keys:
  vm1
  vm2
  Denied Keys:
  Unaccepted Keys:
  Rejected Keys:
  ```

---

