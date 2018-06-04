---
title: Linux日常运维(三)——网络相关命令
author: Evobot
abbrlink: e1f132a7
date: 2018-05-08 21:03:49
categories: Centos7
tags: [Linux, Centos]
image:
---



本文主要介绍Linux网络相关的知识，包括防火墙、路由等方面的配置和用法以及设置DNS，设定IP等。

<!--more-->

---

# Linux网络相关命令

## ifconfig命令

- Centos7默认没有`ifconfig`命令，使用钱需要安装`net-tools`软件包：

  ```bash
  # ifconfig 
  eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 10.139.151.2  netmask 255.255.224.0  broadcast 10.139.159.255
          ether 52:54:00:3f:03:93  txqueuelen 1000  (Ethernet)
          RX packets 4817340  bytes 563694620 (537.5 MiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 5154346  bytes 703940024 (671.3 MiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

  lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
          inet 127.0.0.1  netmask 255.0.0.0
          loop  txqueuelen 1  (Local Loopback)
          RX packets 546  bytes 30132 (29.4 KiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 546  bytes 30132 (29.4 KiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
  ```

- `ifconfig -a`用来在网卡没有ip或者网卡down掉时，同样将网卡信息显示出来。

## ifup/ifdown命令

- `ifdown`用来将网卡状态转为down状态，命令为`ifdown ens33`;

- `ifup`用来将网卡状态置为up状态，命令为`ifup ens33`。

- 在远程连接服务器时，如果对网卡进行了配置，如ip，网关等，不能直接使用`ifdown`关闭网卡，而应该`ifdown ens33 && ifup ens33`这样使用，避免网卡down了之后网络无法恢复。


## 增加虚拟网卡

- 增加虚拟网卡可以为设备增加ip，具体操作为复制原ens33的网卡配置文件：

  ```bash
  # cp /etc/sysconfig/network-scripts/ifcfg-eth0 /etc/sysconfig/network-scripts/ifcfg-eth0\:0
  ```

- 编辑`ifcfg-eth0:0`配置文件：

  ```bash
  BOOTPROTO=none
  DEFROUTE=yes
  # 修改网卡名
  DEVICE=eth0:0
  # 配置网关
  GATEWAY=192.168.199.1
  HWADDR=52:54:00:3f:03:93
  # 配置ip地址及掩码
  IPADDR=192.168.199.133
  NETMASK=255.255.225.0
  NM_CONTROLLED=no
  ONBOOT=yes
  TYPE=Ethernet
  USERCTL=no
  ```

- 保存退出后执行`ifdown eth0 && ifup eth0`命令后网址就会生效：

  ```bash
  [root@evobot network-scripts]# ifconfig 
  eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 10.139.151.2  netmask 255.255.224.0  broadcast 10.139.159.255
          ether 52:54:00:3f:03:93  txqueuelen 1000  (Ethernet)
          RX packets 4820651  bytes 564010062 (537.8 MiB)
          RX errors 0  dropped 0  overruns 0  frame 0
          TX packets 5157474  bytes 704334346 (671.7 MiB)
          TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

  # 新的网卡及地址已生效
  eth0:0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
          inet 192.168.199.133  netmask 255.255.255.0  broadcast 192.168.199.255
          ether 52:54:00:3f:03:93  txqueuelen 1000  (Ethernet)
  ```

## 网卡状态查看

- `mii-tool`命令用来查看网卡是否连接，状态是否正常：

  ```bash
  # mii-tool enp1s0 
  enp1s0: negotiated 1000baseT-FD flow-control, link ok
  ```

- 如果提示**Operation not supported**，也可以使用`ethtool`命令：

  ```bash
  # ethtool enp1s0
  Settings for enp1s0:
  	Supported ports: [ TP MII ]
  	Supported link modes:   10baseT/Half 10baseT/Full 
  	                        100baseT/Half 100baseT/Full 
  	                        1000baseT/Half 1000baseT/Full 
  	Supported pause frame use: No
  	Supports auto-negotiation: Yes
  	Advertised link modes:  10baseT/Half 10baseT/Full 
  	                        100baseT/Half 100baseT/Full 
  	                        1000baseT/Half 1000baseT/Full 
  	Advertised pause frame use: Symmetric Receive-only
  	Advertised auto-negotiation: Yes
  	Link partner advertised link modes:  10baseT/Half 10baseT/Full 
  	                                     100baseT/Half 100baseT/Full 
  	                                     1000baseT/Full 
  	Link partner advertised pause frame use: Symmetric
  	Link partner advertised auto-negotiation: Yes
  	Speed: 1000Mb/s
  	Duplex: Full
  	Port: MII
  	PHYAD: 0
  	Transceiver: internal
  	Auto-negotiation: on
  	Supports Wake-on: pumbg
  	Wake-on: g
  	Current message level: 0x00000033 (51)
  			       drv probe ifdown ifup
  	Link detected: yes	#表示连接状态正常
  ```

## 设置主机名

- Centos7设置主机名使用`hostnamectl`命令，用法为`hostnamectl set-hostname [hostname]`：

  ```bash
  [root@evobot ~]# hostnamectl set-hostname evobot-cn
  [root@evobot ~]# bash	# 新开一个bash生效
  [root@evobot-cn ~]# 
  ```

- hostname的配置文件是`/etc/hostname`：

  ```bash
  [root@evobot-cn ~]# cat /etc/hostname 
  evobot-cn
  ```

## DNS设置

- Centos7的DNS配置在配置文件`/etc/resolv.conf`中：

  ```bash
  [root@evobot ~]# cat /etc/resolv.conf 
  # Your system has been configured with 'manage-resolv-conf' set to true.
  # As a result, cloud-init has written this file with configuration data
  # that it has been provided. Cloud-init, by default, will write this file
  # a single time (PER_ONCE).
  #
  nameserver 100.88.222.14
  nameserver 100.88.222.16	
  ```

- DNS在网卡配置文件中添加DNS即可，临时更改也可以更改`resolv.conf`文件，网卡配置如下：

  ```bash
  TYPE=Ethernet
  BOOTPROTO=static
  DEFROUTE=yes
  PEERDNS=yes
  PEERROUTES=yes
  IPV4_FAILURE_FATAL=no
  IPV6INIT=yes
  IPV6_AUTOCONF=yes
  IPV6_DEFROUTE=yes
  IPV6_PEERDNS=yes
  IPV6_PEERROUTES=yes
  IPV6_FAILURE_FATAL=no
  IPV6_ADDR_GEN_MODE=stable-privacy
  NAME=enp0s3
  UUID=fc2ccabb-fec8-4b8b-a27e-0f5f101310ca
  DEVICE=enp0s3
  ONBOOT=yes
  IPADDR=192.168.199.224
  NETMASK=255.255.255.0
  GATEWAY=192.168.199.1
  DNS1=114.114.114.114
  # 增加一行DNS1，即可为网卡添加一个DNS
  ```

## hosts文件

- Linux中的hosts文件如Windows中相同，主要用来为指定的域名指定对应的ip地址，文件格式左边为ip，右边为域名，且一个ip可以配置多个域名，当有一个域名对应多个ip时，最后面的行才生效：	

  ```bash
  # cat /etc/hosts
  127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
  ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
  192.168.199.224 www.baidu.com

  # ping www.baidu.com
  PING www.baidu.com (192.168.199.224) 56(84) bytes of data.
  64 bytes from www.baidu.com (192.168.199.224): icmp_seq=1 ttl=64 time=0.061 ms
  64 bytes from www.baidu.com (192.168.199.224): icmp_seq=2 ttl=64 time=0.052 ms
  ^C
  --- www.baidu.com ping statistics ---
  2 packets transmitted, 2 received, 0% packet loss, time 999ms
  rtt min/avg/max/mdev = 0.052/0.056/0.061/0.008 ms
  ```

# netfilter防火墙

## 开启netfilter服务

- 临时关闭selinux的命令为`setenforce 0`，永久关闭则是修改`/etc/selinux/config`文件：

  ```bash
  # 修改SELINUX为disabled即可
  SELINUX=disabled
  SELINUXTYPE=targeted
  ```

  ```bash
  $ getenforce 
  Permissive
  # Permissive在遇到安全问题需要阻断时，不会真正阻断，只会进行记录
  ```
  > 关于selinux，可以参考[SELinux：linux内核的安全保镖](http://os.51cto.com/art/201209/355490.htm)一文，或者[下载相应的电子书](https://pan.baidu.com/s/1jGGdExK)。

- netfilter防火墙是Centos7之前的版本默认的防火墙，在Centos7中netfilter是关闭状态的，Centos7中默认的防火墙时firewalld，所以使用netfilter前需要关闭firewalld：

  ```bash
  // 关闭firewalld自启动
  # systemctl disable firewalld.service
  // 停止firewalld服务
  # systemctl stop firewalld.service 
  ```

- 开启netfilter前需要安装`iptables-services`软件包：

  ```bash
  // 安装iptables
  # yum install -y iptables-services
  // 开启iptables服务
  # systemctl start iptables.service
  ```

- 使用命令`ipstables -nvL`可以查看iptables自带的一些规则：

  ```bash
  [root@localhost ~]# iptables -nvL
  Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination         
    135  9408 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
      0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
      0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
      0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
      7  3488 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

  Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination         
      0     0 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

  Chain OUTPUT (policy ACCEPT 72 packets, 7264 bytes)
   pkts bytes target     prot opt in     out     source               destination         
  ```

## netfilter 5表5链

- netfilter默认有5个表(table)，分别是**filter**、**nat** 、**mangle**、**raw**、**security**表；
- filter表主要起过滤数据包作用，包含3个内置的链，分别是**INPUT**、**FORWARD**和**OUTPUT**链：
  - INPUT：用来检查进入本机的数据包，如果发现可疑的源IP，则会禁止掉；
  - FORWARD：是对需要转发的数据包进行处理，会对目的地址非本机的数据包进行转发，或者对目的地址进行更改；
  - OUTPUT：是对本机产生的需要对外发送的数据包进行处理，例如发送给某个IP的数据包，但对方IP被加入到新名单，则OUTPUT会将其禁止。
- nat表中同样具有3个链，分别是**PERROUTING**、**OUTPUT**、**POSTROUTING**，该表主要用在如路由器，端口映射等：
  - PERROUTING：在数据包进来的时候对其进行更改；
  - POSTROUTING：在数据包出去的时候对其进行更改；
  - OUTPUT：与filter表的OUTPUT作用相同。
- 其余的mangle、raw、security表很少使用，这里不做介绍。

## iptables数据处理流程

- iptables的处理流程可以参考下面的流程图：

![iptables流程](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/iptables.png)

  - 当数据包进入网卡时，首先会进入`PERROUTING`链，内核根据数据包目的IP判断是否需要转发；
  - 如果数据包是进入本机的，则会继续到达`INPUT`链，数据包到`INPUT`链后，任何进程都可以收到数据包，本机上的程序可以发送数据包，这些数据包会经过`OUTPUT`链，然后到达`POSTROUTING`链发送出去；
  - 如果数据包不是进入本机的，时需要转发出去的，且内核允许转发，数据包就会进入`FORWARD`链，然后到达`POSTROUTING`链输出。

## iptables语法

### 查看、保存、清空规则

- `iptables -nvL`可以看到默认规则，而默认规则保存在`/etc/sysconfig/iptables`文件内：

  ```bash
  [root@localhost ~]# cat /etc/sysconfig/iptables
  # sample configuration for iptables service
  # you can edit this manually or use system-config-firewall
  # please do not ask us to add additional ports/services to this default configuration
  *filter
  :INPUT ACCEPT [0:0]
  :FORWARD ACCEPT [0:0]
  :OUTPUT ACCEPT [0:0]
  -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
  -A INPUT -p icmp -j ACCEPT
  -A INPUT -i lo -j ACCEPT
  -A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
  -A INPUT -j REJECT --reject-with icmp-host-prohibited
  -A FORWARD -j REJECT --reject-with icmp-host-prohibited
  COMMIT
  ```

- 清空iptables所以规则，使用`iptables -F`，但是这样只能临时清空，实际配置文件中还存在，重启之后会恢复：

  ```bash
  [root@localhost ~]# iptables -F
  [root@localhost ~]# iptables -nvL
  Chain INPUT (policy ACCEPT 31 packets, 2152 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain OUTPUT (policy ACCEPT 16 packets, 1472 bytes)
   pkts bytes target     prot opt in     out     source               destination         
  ```

- 想要保存当前的规则，需要执行`service iptables save`命令；

- 上面的操作所改动的都是`filter`表，想要操作指定表，需要使用`iptables -t [table] -nvL`:

  ```bash
  [root@localhost ~]# iptables -t nat -nvL
  Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination
  ```

- 在查看默认规则时，命令输出有这样两行：

  ```bash
   pkts bytes target     prot opt in     out     source               destination         
     25  1732 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
  ```

  - 这里显示了数据包的数量和字节大小，清空这个数据使用`iptables -Z`将其清零。

### 增加规则

```bash
[root@localhost ~]# iptables -A INPUT -s 192.168.188.1 -p tcp --sport 1234 -d 192.168.188.128 --dport 80 -j DROP
```

{% note danger %}

`-A`表示增加规则，会将规则添加在最后，`INPUT`表示指定为INPUT链，未使用`-t`指定表名，默认操作的是`filter`表，`-s`表示源IP地址，`-p`指定链接的协议是tcp或udp，指定端口时，必须指定协议，`--sport`指定源端口；

`-d`指定目的IP地址，`--dport`指定目的端口，`-j`指定要执行的操作，操作有`DROP`和`REJECT`，DROP表示丢弃，REJECT会在查看数据包之后拒绝并丢弃，一般使用DROP

{% endnote %}

```bash
[root@localhost ~]# iptables -I INPUT -p tcp --dport 80 -j DROP
```

{% note danger %}

这里`-I`表示插入规则，使用`-I`插入规则，规则会在iptables规则的最前面，而`-A`则增加在最后面。

规则的前后，会有限过滤前面的规则，比如一个数据包同时满足两条规则，但会优先匹配在前面的一条，而不会再匹配后面的规则。

{% endnote %}

```bash
[root@localhost ~]# iptables -I INPUT -s 192.168.1.1/24 -i eth0 -j ACCEPT 
```

{% note danger %}

`-i`作用是指定网卡，这里规则的含义是允许来自eth0网卡且源IP为192.168.1.1/24的数据包进入。

{% endnote %}



### 删除规则

```bash
[root@localhost ~]# iptables -D INPUT -p tcp --dport 80 -j DROP
[root@localhost ~]# iptables -D INPUT -s 192.168.188.1 -p tcp --sport 1234 -d 192.168.188.128 --dport 80 -j DROP
```

{% note danger %}

使用`-D`选项可以将之前的规则删除，除了选项为`-D`之外，其他规则内容与增加规则时相同。

{% endnote %}

- 使用`iptables -nvL --line-number`可以在每条规则前输出相应的编号，使用编号可以方便我们删除对应的规则：

  ```bash
  [root@localhost ~]# iptables -nvL --line-number
  Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
  num   pkts bytes target     prot opt in     out     source               destination         
  1      813 57860 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
  2        0     0 ACCEPT     icmp --  *      *       0.0.0.0/0            0.0.0.0/0           
  3        0     0 ACCEPT     all  --  lo     *       0.0.0.0/0            0.0.0.0/0           
  4        0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
  5      191 98123 REJECT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
  6        0     0 DROP       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0  
  ```

  ```bash
  [root@localhost ~]# iptables -D INPUT 6
  # 使用编号对规则进行删除
  ```

### 更改默认策略

- 在iptables输出的规则中，每个链表后面会有一行`policy ACCEPT 0 packets, 0 bytes`，这表示默认策略为ACCEPT，即数据包没有匹配的规则时，就按照默认策略执行；

- 更改默认策略，使用iptables的`-P`选项：

  ```bash
  [root@localhost ~]# iptables -P FORWARD DROP
  Chain FORWARD (policy DROP 0 packets, 0 bytes)
  ```

- 由于更改默认策略容易导致一些问题，一般不建议更改默认策略，保持默认即可。

---