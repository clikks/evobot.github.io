---
title: Docker的安装及镜像管理
author: Evobot
categories: 虚拟化与容器化
tags:
  - Docker
abbrlink: 9431ef69
date: 2019-03-24 21:29:09
image:
---



本文主要介绍一下几点内容：

1. docker简介
2. 安装docker
3. 镜像管理
4. 通过容器创建镜像

<!--more-->

---

# Docker简介

## 什么是Docker

- docker是一个开源的容器引擎，可以让开发者打包应用以及依赖的库，然后发布到任何流行的linux发行版上，移植很方便；
- docker能够实现快速的部署和交付，采用GO语言编写，基于Linux kernel；
- 目前docker分为社区版ce和企业版ee，并且基于年月的时间线形式发布版本；

## Docker和传统虚拟化比较

![docker和传统虚拟化比较](https://s2.ax1x.com/2019/03/24/AYqqtf.png)

- 传统虚拟机技术是虚拟出一套硬件后，在其上运行一个完整操作系统，在该系统上再运行所需应用进程； 
- 而Docker内的应用进程直接运行于宿主的内核，容器内没有自己的内核，而且也没有进行硬件虚拟。因此容器要比传统虚拟机更为轻便，更高效的利用系统资源

对比传统虚拟机总结：

|    特性    |        容器        |   虚拟机   |
| :--------: | :----------------: | :--------: |
|    启动    |        秒级        |   分钟级   |
|  硬盘使用  |      一般为MB      |  一般为GB  |
|    性能    |      接近原生      |    弱于    |
| 系统支持量 | 单机支持上千个容器 | 一般几十个 |

- Docker能够实现一次创建和配置后，可以在任意地方运行；
- 并且是内核级别的虚拟化，不需要额外的hypervisor支持，会有更高的性能和效率；
- Docker易迁移，平台依赖性不强。

## Docker核心概念

- 镜像：镜像是一个只读的模板，类似于安装系统用到的iso文件，通过镜像我们可以完成各种应用的部署；
- 容器：镜像类似于操作系统，而容器类似于虚拟机本身，它可以被启动、开始、停止、删除等操作，每个容器都是相互隔离的；
- 仓库：仓库是存放镜像的一个场所，仓库分为公开仓库和私有仓库，最大的公开仓库是[Docker Hub](hub.docker.com).

# 安装Docker

- 安装Docker首先安装所需的`yum-utils`、`device-mapper-persistent-data`和`lvm2`包：

  ```bash
  yum install -y yum-utils device-mapper-persistent-data lvm2
  ```

- 然后安装官方提供的yum源，使用`yum-utils`软件包提供的`yum-config-manager`命令添加新的软件仓库：

  ```bash
  yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
  ```

- 安装完成后可以查看仓库内提供的docker版本：

  ```bash
  [root@localhost nginx_config]# yum list |grep docker
  cockpit-docker.x86_64                      176-4.el7.centos            extras
  containerd.io.x86_64                       1.2.4-3.1.el7               docker-ce-stable
  docker.x86_64                              2:1.13.1-94.gitb2f74b2.el7.centos
  docker-ce.x86_64                           3:18.09.3-3.el7             docker-ce-stable
  docker-ce-cli.x86_64                       1:18.09.3-3.el7             docker-ce-stable
  docker-ce-selinux.noarch                   17.03.3.ce-1.el7            docker-ce-stable
  docker-client.x86_64                       2:1.13.1-94.gitb2f74b2.el7.centos
  docker-client-latest.x86_64                1.13.1-58.git87f2fab.el7.centos
  docker-common.x86_64                       2:1.13.1-94.gitb2f74b2.el7.centos
  docker-distribution.x86_64                 2.6.2-2.git48294d9.el7      extras
  docker-latest.x86_64                       1.13.1-58.git87f2fab.el7.centos
  docker-latest-logrotate.x86_64             1.13.1-58.git87f2fab.el7.centos
  docker-latest-v1.10-migrator.x86_64        1.13.1-58.git87f2fab.el7.centos
  docker-logrotate.x86_64                    2:1.13.1-94.gitb2f74b2.el7.centos
  docker-lvm-plugin.x86_64                   2:1.13.1-94.gitb2f74b2.el7.centos
  docker-novolume-plugin.x86_64              2:1.13.1-94.gitb2f74b2.el7.centos
  docker-registry.x86_64                     0.9.1-7.el7                 extras
  docker-v1.10-migrator.x86_64               2:1.13.1-94.gitb2f74b2.el7.centos
  pcp-pmda-docker.x86_64                     4.1.0-5.el7_6               updates
  podman-docker.noarch                       0.12.1.2-2.git9551f6b.el7.centos
  python-docker-py.noarch                    1:1.10.6-8.el7_6            extras
  python-docker-pycreds.noarch               1:0.3.0-8.el7_6             extras
  
  ```

  可以看到centos的基础仓库也提供了docker的软件包，但版本太老旧，而官方docker-ce-stable仓库的版本已经是18.09版本，所以我们安装官方仓库的docker软件包。

- 使用`yum install -y docker-ce`安装docker即可，国内安装速度会比较慢，也可以[下载rpm包](<https://download.docker.com/linux/centos/7/x86_64/stable/Packages/>)之后使用yum安装rpm包。

- 安装完成后使用`systemctl start docker`启动docker，docker启动时会在ipstables中添加几条防火墙规则，也可以使用`service iptables save`将docker添加的规则保存：

  ```bash
  [root@localhost ~]# service iptables save
  iptables: Saving firewall rules to /etc/sysconfig/iptables:[  确定  ]
  
  ```


# Docker镜像管理

## 加速器配置

- docker的镜像概念上类似于安装系统使用的iso文件，默认docker下载镜像是从国外的官网下载，速度很慢；

- 使用`docker pull [镜像名]`就可以从docker的官方仓库下载镜像；

- 为了提高下载镜像的速度，我们可以为docker配置加速器，编辑`/etc/docker/daemon.json`配置文件（没有就新建一个），添加下面的配置，这里使用了[daocloud](https://www.daocloud.io/mirror)的加速器：

  ```json
  {
      "registry-mirrors": ["http://f1361db2.m.daocloud.io"]
  }
  ```

- 配置完加速器后，重启docker服务，再下载镜像就会从加速器的地址进行下载。

## 镜像管理命令

- 使用`docker pull centos`下载centos镜像：

  ```bash
  [root@localhost /]# docker pull centos
  Using default tag: latest
  latest: Pulling from library/centos
  8ba884070f61: Pull complete
  Digest: sha256:8d487d68857f5bc9595793279b33d082b03713341ddec91054382641d14db861
  Status: Downloaded newer image for centos:latest
  
  ```

- `docker images`命令可以查看本地的镜像列表：

  ```bash
  [root@localhost /]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              9f38484d220f        2 weeks ago         202MB
  
  ```

  这里TAG是指镜像的标签，IMAGE ID用来区分image的唯一标识。

- docker的pull命令类似于git的pull，如果搜索镜像，可以使用`docker  search [镜像名]`查看仓库有哪些对应的镜像：

  ```bash
  [root@localhost /]# docker search jumpserver
  NAME                             DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
  jumpserver/jumpserver                                                            14
  jiaxiangkong/jumpserver_docker   开源跳板机(堡垒机):认证,授权,审计,自动化运维                       10
  jumpserver/jms_all                                                               6
  wojiushixiaobai/jumpserver       Jumpserver ALL                                  5                   [OK]
  hhding/jumpserver-docker         ssh proxy node                                  3                   [OK]
  njqaaa/jumpserver                jumpserver                                      2                   [OK]
  jumpserver/guacamole             guacamole for jumpserver                        2                   [OK]
  
  ```

  所以我们也可以自己制作镜像上传到仓库中供其他人下载使用。

- `docker tag [镜像名] [标签名]`命令可以给镜像打标签，打了标签之后，就会生成一个新的镜像：

  ```bash
  [root@localhost /]# docker tag centos evobot_centos
  [root@localhost /]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              9f38484d220f        2 weeks ago         202MB
  evobot_centos       latest              9f38484d220f        2 weeks ago         202MB
  
  ```

  - 可以看到新生成的镜像名，但实际上从`IMAGE ID`上可以看出来，这两个镜像实际上是相同的。
  - 而新生成的镜像中`TAG`也是latest，如果想要改变这个TAG，则可以使用`docker tag [镜像名] [新镜像名:新TAG]`：

  ```bash
  [root@localhost ~]# docker tag centos evobot_centos2:test1
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              9f38484d220f        2 weeks ago         202MB
  evobot_centos2      test1               9f38484d220f        2 weeks ago         202MB
  evobot_centos       latest              9f38484d220f        2 weeks ago         202MB
  [root@localhost ~]# docker tag centos centos:test2
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              9f38484d220f        2 weeks ago         202MB
  centos              test2               9f38484d220f        2 weeks ago         202MB
  evobot_centos2      test1               9f38484d220f        2 weeks ago         202MB
  evobot_centos       latest              9f38484d220f        2 weeks ago         202MB
  
  ```

  - 上面的输出结果可以看到，镜像名和TAG都可以单独修改。

- 将镜像启动为容器，使用`docker run -itd [镜像名]`，其中`-i`表示让容器的标准输入打开，`-t`表示分配一个伪终端，`-d`表示后台启动，这几个选项必须放在镜像名字前面，启动容器后只会输出容器的ID:

  ```bash
  [root@localhost ~]# docker run -itd centos
  95af29cb769e67579f27be9af40b5c92fa8d766900df9725b30299d4cd878ba6
  
  ```

- `docker ps`可以查看正在运行的容器，容器的状态可以是运行状态，也可以是停止状态，而`docker ps`只能查看运行状态的容易，要查看所有状态的容器，使用`docker ps -a`命令：

  ```bash
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
  95af29cb769e        centos              "/bin/bash"         About a minute ago   Up About a minute                   nifty_wescoff
  
  ```

- 删除一个镜像，使用`docker rmi [镜像名:TAG]`,在不指定TAG的情况下，docker默认会删除TAG为latest的镜像，如果没有latest的镜像就会报错查找不到镜像：

  ```bash
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              9f38484d220f        2 weeks ago         202MB
  centos              test2               9f38484d220f        2 weeks ago         202MB
  evobot_centos2      test1               9f38484d220f        2 weeks ago         202MB
  evobot_centos       latest              9f38484d220f        2 weeks ago         202MB
  
  [root@localhost ~]# docker rmi evobot_centos2
  Error: No such image: evobot_centos2
  
  [root@localhost ~]# docker rmi evobot_centos2:test1
  Untagged: evobot_centos2:test1
  
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              9f38484d220f        2 weeks ago         202MB
  centos              test2               9f38484d220f        2 weeks ago         202MB
  evobot_centos       latest              9f38484d220f        2 weeks ago         202MB
  
  [root@localhost ~]# docker rmi evobot_centos
  Untagged: evobot_centos:latest
  
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              latest              9f38484d220f        2 weeks ago         202MB
  centos              test2               9f38484d220f        2 weeks ago         202MB
  
  ```

- 上面的删除操作实际上只是删除了镜像的TAG，类似于删除Linux中的硬链接，而要彻底删除镜像，则需要在删除时指定镜像的`IMAGE ID`，这时会删除整个镜像，包括该依赖该镜像创建的标签也会一起被删除：

  ```bash
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos              test2               9f38484d220f        2 weeks ago         202MB
  [root@localhost ~]# docker rmi 9f38484d220f
  Untagged: centos:test2
  Untagged: centos@sha256:8d487d68857f5bc9595793279b33d082b03713341ddec91054382641d14db861
  Deleted: sha256:9f38484d220fa527b1fb19747638497179500a1bed8bf0498eb788229229e6e1
  Deleted: sha256:d69483a6face4499acb974449d1303591fcbb5cdce5420f36f8a6607bda11854
  
  ```

  - 需要注意，删除镜像时，必须先停止正在运行的容器，使用`doker stop [容器ID]`停止容器，然后再执行删除镜像操作，如果一个镜像存在多个TAG的时候，需要在删除时先指定镜像名删除，然后再使用IMAGE ID删除镜像。

# 通过容器创建镜像

- 镜像除了通过官方仓库拉取之外，还可以根据自己的需要，在镜像中安装所需的软件后，创建定制化的镜像；

- 首先使用`docker run -itd centos`启动容器，再使用`docker exec -it [CONTAINER ID] bash`命令可以进入容器的命令行：

  ```bash
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  b27af36dca0b        centos              "/bin/bash"         2 minutes ago       Up 2 minutes                            reverent_joliot
  [root@localhost ~]# docker exec -it b27af36dca0b bash
  [root@b27af36dca0b /]#
  
  ```

  可以看到命令行的提示符已经变成了CONTAINER ID，说明已经进入了容器的命令行。

- 进入容器后，我们查看容器的内存、磁盘可以看到所有的信息都与宿主机相同，我们可以尝试使用yum安装net-tools软件，如果发现容器内无法联网，那么需要推出到宿主机，编辑`/etc/sysctl.conf`文件，在文件内增加一行内容如下：

  ```ini
  net.ipv4.ip_forward=1
  ```

  然后重启网络服务，使用`sysctl net.ipv4.ip_forward`命令查看修改是否生效，生效后再进入容器就可以联网了。

- 在容器内执行`ifconfig`命令可以看到容器的IP是一个docker默认的网段，并且在宿主机上，也会多出一个`docker0`的虚拟网卡。

  ```bash
  [root@b27af36dca0b /]# ifconfig
  eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 172.17.0.2  netmask 255.255.0.0  broadcast 0.0.0.0
          ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
          RX packets 2874  bytes 10671258 (10.1 MiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 2422  bytes 134408 (131.2 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  [root@b27af36dca0b /]# exit
  exit
  [root@localhost ~]# ifconfig
  docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
          inet6 fe80::42:13ff:fe60:4f56  prefixlen 64  scopeid 0x20<link>
          ether 02:42:13:60:4f:56  txqueuelen 0  (Ethernet)
          RX packets 2437  bytes 101450 (99.0 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 2888  bytes 10675146 (10.1 MiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  ```

- 接下来我们将这个安装了net-tools的容器做成镜像，在容器内输入`exit`或者`ctrl+d`退出容器命令行，然后使用`docker commit -m [DESCRIPTION] -a [IMAGE AUTHOR] [container ID] [new container name]`命令创建镜像：

  ```bash
  [root@localhost ~]# docker commit -m "install net-tools" -a "evobot" b27af36dca0b centos_with_net
  sha256:52992c2b54875c47ffe7347424016b88286d0c5b5a7ce316662eb9f31f1217e2
  
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
  centos_with_net     latest              52992c2b5487        About a minute ago   285MB
  centos              latest              9f38484d220f        3 weeks ago          202MB
  ```

- 新生成的镜像可以看到IMAGE ID已经是新的ID，并且大小也与之前的centos镜像不同，然后就可以使用新的镜像来创建容器：

  ```bash
  [root@localhost ~]# docker run -itd centos_with_net bash
  fc0e7fef408945355b85d482a80def8aca37bf661c401669da9f79c87270dcb5
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  fc0e7fef4089        centos_with_net     "bash"              48 seconds ago      Up 46 seconds                           wonderful_gates
  b27af36dca0b        centos              "/bin/bash"         About an hour ago   Up About an hour                        reverent_joliot
  [root@localhost ~]# docker exec -it wonderful_gates bash
  [root@fc0e7fef4089 /]# ifconfig
  eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 172.17.0.3  netmask 255.255.0.0  broadcast 0.0.0.0
          ether 02:42:ac:11:00:03  txqueuelen 0  (Ethernet)
          RX packets 8  bytes 648 (648.0 B)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 0  bytes 0 (0.0 B)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
          inet 127.0.0.1  netmask 255.0.0.0
          loop  txqueuelen 1000  (Local Loopback)
          RX packets 0  bytes 0 (0.0 B)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 0  bytes 0 (0.0 B)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  ```

  这里进入容器使用了容器的名字，容器名是可以自定义的，并且新运行的容器也会在宿主机上创建一个新的虚拟网卡：

  ```bash
  [root@localhost ~]# ifconfig
  docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 172.17.0.1  netmask 255.255.0.0  broadcast 0.0.0.0
          inet6 fe80::42:13ff:fe60:4f56  prefixlen 64  scopeid 0x20<link>
          ether 02:42:13:60:4f:56  txqueuelen 0  (Ethernet)
          RX packets 2437  bytes 101450 (99.0 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 2931  bytes 10689852 (10.1 MiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 192.168.139.128  netmask 255.255.255.0  broadcast 192.168.139.255
          inet6 fe80::44d4:1853:c84:4437  prefixlen 64  scopeid 0x20<link>
          ether 00:0c:29:84:90:4c  txqueuelen 1000  (Ethernet)
          RX packets 232575  bytes 296239214 (282.5 MiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 67561  bytes 14709339 (14.0 MiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
          inet 127.0.0.1  netmask 255.0.0.0
          inet6 ::1  prefixlen 128  scopeid 0x10<host>
          loop  txqueuelen 1000  (Local Loopback)
          RX packets 24  bytes 2112 (2.0 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 24  bytes 2112 (2.0 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  veth66eba1f: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet6 fe80::6410:d9ff:fe8c:103  prefixlen 64  scopeid 0x20<link>
          ether 66:10:d9:8c:01:03  txqueuelen 0  (Ethernet)
          RX packets 0  bytes 0 (0.0 B)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 10  bytes 1332 (1.3 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  vethf92e98c: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet6 fe80::540f:80ff:fe45:5466  prefixlen 64  scopeid 0x20<link>
          ether 56:0f:80:45:54:66  txqueuelen 0  (Ethernet)
          RX packets 2422  bytes 134408 (131.2 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 2917  bytes 10685964 (10.1 MiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  
  ```

---