---
title: Docker数据卷备份恢复及网络配置
author: Evobot
date: 2019-04-11 21:58:21
categories: 虚拟化与容器化
tags:
  - Docker
image:
---

1. 数据卷备份恢复
2. docker网络模式
3. 配置桥接网络

<!--more-->

---

# 数据卷的备份恢复

## 数据卷备份

- 对于做了宿主机目录映射的数据卷，其实直接备份服务器磁盘即可，但docker创建数据卷容器时可以不做本地目录映射，这种情况就需要对数据卷进行备份；

- 首先在宿主机磁盘上创建一个目录，例如`/vol_data_backup`:

  ```bash
  [root@localhost ~]# mkdir /vol_data_backup
  ```

- 然后需要再新建一个用来备份的容器，这个容器要挂载需要备份的数据卷，同时还要将上面创建的宿主机目录，映射到容器内去：

  ```bash
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS
  36f577a6cf19        centos              "bash"                   2 days ago          Up 3 minutes
  6f9aedc18569        registry            "/entrypoint.sh /e..."   2 days ago          Up 3 minutes        0.0.0.0:5000->5000/tcp
  
  [root@localhost ~]# docker run -itd --volumes-from testvol -v /vol_data_backup/:/backup centos bash
  462d264a2605e9e24973822071e95dd7ec040049fc7b2bb84e55f9ac2b107233
  
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS   NAMES
  462d264a2605        centos              "bash"                   6 seconds ago       Up 5 seconds   happy_kowalevski
  36f577a6cf19        centos              "bash"                   2 days ago          Up 4 minutes   testvol
  6f9aedc18569        registry            "/entrypoint.sh /e..."   2 days ago          Up 4 minutes        0.0.0.0:5000->5000/tcp   zealous_kilby
  
  ```

- 经过上面创建容器的步骤，我们创建的新容器内即挂载了需要备份的数据卷，又映射了本地目录，然后只需要进入容器，将要备份的数据卷打包保存到本地映射到容器内的`/backup`目录即可：

  ```bash
  [root@localhost ~]# docker exec -it happy_kowalevski bash
  
  [root@462d264a2605 /]# tar -cvf /backup/data.tar data/
  data/
  
  ```

- 这样我们就可以在本地宿主机上的`/vol_data_backup`目录内看到备份的打包文件：

  ```bash
  [root@localhost ~]# cd /vol_data_backup/
  [root@localhost vol_data_backup]# ls
  data.tar
  
  ```

## 数据卷恢复

- 恢复数据卷的思路和备份相同，需要先新建一个数据卷容器，然后再新建一个挂载数据卷并映射本地备份目录的容器，然后进入容器后，将映射目录内的备份解压到数据卷目录即可，步骤如下：

  ```bash
  [root@localhost vol_data_backup]# docker run -itd -v /data --name recovery_vol centos bash
  4d2501097084198964a6b34eb81b7bb5b06d29830cfba3674663b9411738d1ca
  
  [root@localhost vol_data_backup]# docker run -itd --volumes-from recovery_vol -v /vol_data_backup/:/recovery centos bash
  b2c06cc0d74e227a42b36e00203ddf2d363afdfa5a9c3130e935c9a12afa277f
  
  [root@localhost vol_data_backup]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS    NAMES
  b2c06cc0d74e        centos              "bash"                   25 seconds ago       Up 25 seconds    compassionate_engelbart
  4d2501097084        centos              "bash"                   About a minute ago   Up About a minute    recovery_vol
  
  [root@localhost vol_data_backup]# docker exec -it compassionate_engelbart bash
  [root@b2c06cc0d74e /]# ls
  anaconda-post.log  bin  data  dev  etc  home  lib  lib64  media  mnt  opt  proc  recovery  root  run  sbin  srv  sys  tmp  usr
  [root@b2c06cc0d74e /]# cd /recovery/
  [root@b2c06cc0d74e recovery]# ls
  data.tar
  
  [root@b2c06cc0d74e recovery]# tar xvf data.tar -C /data/
  data/
  
  ```

# Docker的网络模式

## Docker网络模式介绍

- Docker一共有四种网络模式，分别是host，container，none，bridge四种；
- host模式，使用`docker run`时，使用`--net=host`参数指定，这种模式，容器的网络和宿主机相同，容器内看到的IP就是宿主机的IP；
- container模式，使用`--net=container:container_id/container_name`参数，后面指定容器ID或者容器名都可以，这种模式是多个容器使用共同的网络，多个容器内看到的网卡IP是相同的；
- none模式，使用`--net=none`选项指定，这个模式下，容器不会配置任何网络；
- bridge模式，使用`--net=bridge`选项指定，该模式是docker默认的模式，不指定网络模式时容器都是采用bridge模式，bridge模式会为每个容器分配一个独立的Network Namespace，类似于vmware的nat网络模式，同一个宿主机上的所有容器都会在同一个网段下，相互之间也可以通信，容器内也能够联网。

## 外部访问容器

- docker启动一个容器，在不指定网络模式的情况下，会默认使用bridge网络模式，这种模式下，只有宿主机能够和容器通信，外部机器无法访问容器；

- 为了让外部机器能够访问容器内的服务，在创建容器时，使用`-p [host_port]:[container_port]`选项将容器内的端口映射到宿主机上;

- 这里在centos基础镜像创建的容器内安装httpd软件包，然后将容器导出为镜像：

  ```bash
  [root@localhost ~]# docker run -itd centos bash
  489208280a4476ecbbda0ee230b253b6e4f9bd61588579a698aed28922435193
  
  [root@localhost ~]# docker exec -it 4892 bash
  [root@489208280a44 /]# yum install httpd -y
  
  [root@489208280a44 /]# exit
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  489208280a44        centos              "bash"              3 minutes ago       Up 3 minutes                            youthful_panini
  
  [root@localhost ~]# docker commit -m "install httpd" -a "evobot" 489208280a44 centos_with_httpd
  sha256:a2efe3f26cf59a1d7ea8b0186977d74b153d8a9241e3396dcc5b0a7c074e3df6
  [root@localhost ~]# docker images
  REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
  centos_with_httpd              latest              a2efe3f26cf5        4 seconds ago       318MB
  centos                         latest              9f38484d220f        3 weeks ago         202MB
  
  ```

- 然后使用安装了httpd的镜像创建一个新的容器，并将80端口映射到宿主机上：

  ```bash
  [root@localhost ~]# docker run -itd -p 8088:80 centos_with_httpd
  b54bbbc6d990a9917620d4c726680313296aa88cb31f6ec0c2b3fd9644b50dfb
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
  b54bbbc6d990        centos_with_httpd   "bash"              2 seconds ago       Up 2 seconds        0.0.0.0:8088->80/tcp   sleepy_jackson
  489208280a44        centos              "bash"              8 minutes ago       Up 8 minutes                               youthful_panini
  
  ```

- 接着进入新创建的容器，在容器中启动httpd服务：

  ```bash
  [root@localhost ~]# docker exec -it b54bbbc6d990 bash
  [root@b54bbbc6d990 /]# systemctl start httpd
  Failed to get D-Bus connection: Operation not permitted
  
  ```

- 在容器内启动httpd服务时报错`Operation not permitted`，这是由于容器内的`dbus-daemon`服务没有启动，所以在容器内启动服务时会报错权限不足，解决这个问题，需要在创建容器时，增加`--privileged -e "container=docker"`参数，并且最后的命令不再使用`bash`，而是使用`/usr/sbin/init`：

  ```bash
  [root@localhost ~]# docker rm -f b54bbbc6d990
  b54bbbc6d990
  
  [root@localhost ~]# docker run -itd --privileged -e "container=docker" --name=httpd -p 8088:80 centos_with_httpd /usr/sbin/init
  b69cb1bcec238a16b728c1c88fc6129096d6e826422f19dab599a9d74f8e9f18
  
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS                  NAMES
  b69cb1bcec23        centos_with_httpd   "/usr/sbin/init"    10 seconds ago      Up 9 seconds        0.0.0.0:8088->80/tcp   httpd
  489208280a44        centos              "bash"              15 minutes ago      Up 15 minutes                              youthful_panini
  
  ```

- 然后重新进入容器，启动httpd服务：

  ```bash
  [root@localhost ~]# docker exec -it httpd bash
  
  [root@b69cb1bcec23 /]# systemctl start httpd
  
  [root@b69cb1bcec23 /]# ps aux |grep httpd
  root       3388  0.8  1.0 224052  4976 ?        Ss   15:51   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     3389  0.0  0.6 224052  2944 ?        S    15:51   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     3390  0.0  0.6 224052  2944 ?        S    15:51   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     3391  0.0  0.6 224052  2944 ?        S    15:51   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     3392  0.0  0.6 224052  2944 ?        S    15:51   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     3393  0.0  0.6 224052  2944 ?        S    15:51   0:00 /usr/sbin/httpd -DFOREGROUND
  root       3395  0.0  0.1   9088   672 pts/1    S+   15:51   0:00 grep --color=auto httpd
  
  [root@b69cb1bcec23 /]# curl localhost
  ```

- 退出容器，在宿主机和外部机器上尝试访问8088端口的httpd服务：

  ```bash
  #宿主机上访问
  [root@localhost ~]# curl localhost:8088
  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd"><html><head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
                  <title>Apache HTTP Server Test Page powered by CentOS</title>
                  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  
      <!-- Bootstrap -->
      <link href="/noindex/css/bootstrap.min.css" rel="stylesheet">
      <link rel="stylesheet" href="noindex/css/open-sans.css" type="text/css" />
  
  <style type="text/css"><!--
  ...
  ```

  ```bash
  #在外部机器访问
  [root@vm2 ~]# curl 192.168.139.128:8088
  <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN" "http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd"><html><head>
  <meta http-equiv="content-type" content="text/html; charset=UTF-8">
                  <title>Apache HTTP Server Test Page powered by CentOS</title>
                  <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
  
      <!-- Bootstrap -->
      <link href="/noindex/css/bootstrap.min.css" rel="stylesheet">
      <link rel="stylesheet" href="noindex/css/open-sans.css" type="text/css" />
  ...
  ```

- 可以看到外部机器和宿主机都能够通过映射的端口访问到容器内的httpd服务。

# 桥接网络配置

## 宿主机网络配置

- 为了使本地网络中的机器和Docker容器更方便通信，我们经常会有将Docker容器配置道和主机同一网段的需求，实现这个需求，只需要将Docker容器和宿主机的网卡桥接起来，再给Docker容器配上IP即可；

- 桥接后Docker容器可以让其他机器访问，也可以安装sshd服务，让远程用户登录到Docker容器中；

- 首先进入宿主机的网卡配置目录，将网卡配置复制一份并重命名为`ifcfg-br0`:

  ```bash
  [root@localhost ~]# cd /etc/sysconfig/network-scripts/
  [root@localhost network-scripts]# ls
  ifcfg-ens33  ifdown-eth   ifdown-post    ifdown-Team      ifup-aliases  ifup-ipv6   ifup-post    ifup-Team      init.ipv6-global
  ifcfg-lo     ifdown-ippp  ifdown-ppp     ifdown-TeamPort  ifup-bnep     ifup-isdn   ifup-ppp     ifup-TeamPort  network-functions
  ifdown       ifdown-ipv6  ifdown-routes  ifdown-tunnel    ifup-eth      ifup-plip   ifup-routes  ifup-tunnel    network-functions-ipv6
  ifdown-bnep  ifdown-isdn  ifdown-sit     ifup             ifup-ippp     ifup-plusb  ifup-sit     ifup-wireless
  [root@localhost network-scripts]# cp ifcfg-ens33 ifcfg-br0
  
  ```

- 然后编辑`ifcfg-br0`，将网卡类型修改为`Bridge`，`NAME`和`DEVICE`改为`br0`，其他配置不变，如下：

  ```ini
  TYPE=Bridge
  PROXY_METHOD=none
  BROWSER_ONLY=no
  BOOTPROTO=static
  DEFROUTE=yes
  IPV4_FAILURE_FATAL=no
  IPV6INIT=yes
  IPV6_AUTOCONF=yes
  IPV6_DEFROUTE=yes
  IPV6_FAILURE_FATAL=no
  IPV6_ADDR_GEN_MODE=stable-privacy
  NAME=br0
  UUID=4a9dd428-04ef-436f-98ef-16d7cdf3a48b
  DEVICE=br0
  ONBOOT=yes
  DNS1=114.114.114.114
  IPADDR=192.168.139.128
  PREFIX=24
  GATEWAY=192.168.139.2
  
  
  ```

- 然后编辑宿主机网卡配置文件`ifcfg-ens33`，注释掉网卡的UUID以及IP配置，将`BOOTPROTO`设置为`none`，并添加`Bridge`配置项，如下：

  ```ini
  TYPE=Ethernet
  PROXY_METHOD=none
  BROWSER_ONLY=no
  BOOTPROTO=none
  DEFROUTE=yes
  IPV4_FAILURE_FATAL=no
  IPV6INIT=yes
  IPV6_AUTOCONF=yes
  IPV6_DEFROUTE=yes
  IPV6_FAILURE_FATAL=no
  IPV6_ADDR_GEN_MODE=stable-privacy
  NAME=ens33
  #UUID=4a9dd428-04ef-436f-98ef-16d7cdf3a48b
  DEVICE=ens33
  ONBOOT=yes
  #DNS1=114.114.114.114
  #IPADDR=192.168.139.128
  #PREFIX=24
  #GATEWAY=192.168.139.2
  BRIDGE=br0
  
  ```

- 上面的配置相当于将ens33的IP配置到了br0上，而ens33将br0作为桥接的对象，完成配置后，使用`systemctl restart network`重启网络，配置正确的话，网络仍然是联通的，如果配置错误，重启服务后，网络会断开：

  ```bash
  [root@localhost ~]# systemctl restart network
  [root@localhost ~]# ifconfig
  br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 192.168.139.128  netmask 255.255.255.0  broadcast 192.168.139.255
          inet6 fe80::4ddc:2494:3689:5411  prefixlen 64  scopeid 0x20<link>
          ether 00:0c:29:84:90:4c  txqueuelen 1000  (Ethernet)
          RX packets 17  bytes 1247 (1.2 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 19  bytes 1700 (1.6 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
          inet6 fe80::42:b5ff:fe15:e60b  prefixlen 64  scopeid 0x20<link>
          ether 02:42:b5:15:e6:0b  txqueuelen 0  (Ethernet)
          RX packets 1749  bytes 131689 (128.6 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 2842  bytes 35888741 (34.2 MiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          ether 00:0c:29:84:90:4c  txqueuelen 1000  (Ethernet)
          RX packets 31664  bytes 37710668 (35.9 MiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 5693  bytes 680708 (664.7 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  
  ```

  重启网络服务后，正确的情况ens33是没有IP地址的，而IP地址则在br0上。

## 容器网络桥接

- 完成了宿主机的网络配置后，需要安装`pipework`软件包，直接从[pipework的github仓库](https://github.com/jpetazzo/pipework.git)克隆即可:

  ```bash
  [root@localhost ~]# git clone https://github.com/jpetazzo/pipework.git
  正克隆到 'pipework'...
  remote: Enumerating objects: 501, done.
  remote: Total 501 (delta 0), reused 0 (delta 0), pack-reused 501
  接收对象中: 100% (501/501), 172.97 KiB | 115.00 KiB/s, done.
  处理 delta 中: 100% (264/264), done.
  
  ```

- 然后将pipework目录内的`pipework`可执行文件，复制到PATH环境变量所指定的目录内，这样就可以直接执行`pipework`命令了：

  ```bash
  [root@localhost ~]# cd pipework/
  [root@localhost pipework]# ls -l
  总用量 60
  -rw-r--r--. 1 root root    75 4月  13 01:26 docker-compose.yml
  drwxr-xr-x. 2 root root    24 4月  13 01:26 doctoc
  -rw-r--r--. 1 root root 11358 4月  13 01:26 LICENSE
  -rwxr-xr-x. 1 root root 14698 4月  13 01:26 pipework
  -rw-r--r--. 1 root root   827 4月  13 01:26 pipework.spec
  -rw-r--r--. 1 root root 22328 4月  13 01:26 README.md
  [root@localhost pipework]# cp pipework /usr/local/bin/
  [root@localhost pipework]# pipework
  Syntax:
  pipework <hostinterface> [-i containerinterface] [-l localinterfacename] [-a addressfamily] <guest> <ipaddr>/<subnet>[@default_gateway] [macaddr][@vlan]
  pipework <hostinterface> [-i containerinterface] [-l localinterfacename] <guest> dhcp [macaddr][@vlan]
  pipework route <guest> <route_command>
  pipework rule <guest> <rule_command>
  pipework tc <guest> <tc_command>
  pipework --wait [-i containerinterface]
  
  ```

- 完成后我们运行一个不配置网络的容器：

  ```bash
  [root@localhost pipework]# docker run -itd --net=none --name=web --privileged -e "container=docker" centos_with_httpd /usr/sbin/init
  9eb9ce28eb78157ff0ecb63c45986cd1d8fa88f8a6f68de5efbc97df9491f218
  
  ```

- 运行容器后，使用`pipework`命令为容器配置一个桥接的网络，命令格式为`pipework br0 [container_id|container_name] [ip/prefix@gateway]`，其中，ip是分配给容器的ip，@后面为网关地址：

  ```bash
  [root@localhost pipework]# pipework br0 web 192.168.139.200/24@192.168.139.2
  [root@localhost pipework]# docker exec -it web bash
  [root@9eb9ce28eb78 /]# ifconfig
  eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 192.168.139.200  netmask 255.255.255.0  broadcast 192.168.139.255
          ether 5a:d4:79:bd:58:08  txqueuelen 1000  (Ethernet)
          RX packets 310  bytes 349233 (341.0 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 273  bytes 17112 (16.7 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  
  ```

  可以看到，容器已经有了使用`pipework`指定的IP地址，并且从其他机器上也能够ping通容器的地址，启动httpd服务后也可以直接访问80端口：

  ```bash
  [root@vm2 ~]# ping 192.168.139.200
  PING 192.168.139.200 (192.168.139.200) 56(84) bytes of data.
  64 bytes from 192.168.139.200: icmp_seq=1 ttl=64 time=0.365 ms
  64 bytes from 192.168.139.200: icmp_seq=2 ttl=64 time=0.408 ms
  64 bytes from 192.168.139.200: icmp_seq=3 ttl=64 time=0.421 ms
  ^C
  --- 192.168.139.200 ping statistics ---
  3 packets transmitted, 3 received, 0% packet loss, time 2000ms
  rtt min/avg/max/mdev = 0.365/0.398/0.421/0.023 ms
  
  [root@vm2 ~]# nmap 192.168.139.200 -p 80
  
  Starting Nmap 6.40 ( http://nmap.org ) at 2019-04-13 01:48 CST
  Nmap scan report for 192.168.139.200
  Host is up (0.00047s latency).
  PORT   STATE SERVICE
  80/tcp open  http
  MAC Address: 5A:D4:79:BD:58:08 (Unknown)
  
  Nmap done: 1 IP address (1 host up) scanned in 0.06 seconds
  
  ```

- 这样，我们就完成了容器网络桥接的配置，而docker官方的Bridge模式，其实和虚拟机的NAT模式相同。

---

