title: Centos7系列:系统网络配置
author: Evobot
abbrlink: 144d2216
date: 2018-03-21 01:36:54
categories: Centos7
tags: [Linux, Centos]
image: http://p5qynomrl.bkt.clouddn.com/15215702249616hbmgtp3.png?imageslim
---
开启虚拟机，使用root账户登录到Centos中，因为我们在创建虚拟机的时候，使用的是nat模式，所以这个时候系统其实是可以使用dhcp上网的，但是有时候我们还需要配置静态IP等，这时候就需要对系统网络进行配置。
<!-- more -->
# DHCP配置
## 虚拟网络编辑器
- 虚拟机刚运行时，系统并不会开启DHCP服务，所以这时可以运行`dhclient`命令让系统自动获取ip地址：
![dhclient](http://p5qynomrl.bkt.clouddn.com/1521568317622jtndcbms.png?imageslim)
- 可以图上看到系统获取的ip地址为**192.168.253.129**，这个地址其实是VMware的虚拟网络编辑器中配置的，点击VMware菜单栏的编辑-虚拟网络编辑器，可以看到NAT模式所配置的IP地址段：
![虚拟网络编辑器](http://p5qynomrl.bkt.clouddn.com/1521568518435iqk6akyj.png?imageslim)
- 点击**DHCP**设置，即可对虚拟网络的IP地址段进行配置：
![DHCP配置](http://p5qynomrl.bkt.clouddn.com/1521568591065reso26ik.png?imageslim)

# 静态IP配置
## 网卡配置文件
- Centos的IP配置文件是`/etc/sysconfig/network-scripts/`目录下，以`ifcfg-`开头，网络端口名结尾的文件，上面看到端口名为`ens33`，所以我们需要编辑这个文件：
```bash
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:71:8a:c0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.253.129/24 brd 192.168.253.255 scope global dynamic ens33
       valid_lft 785sec preferred_lft 785sec
    inet6 fe80::20c:29ff:fe71:8ac0/64 scope link
       valid_lft forever preferred_lft forever
```

## 配置静态IP
- 使用`vi`命令编辑这个文件`vi /etc/sysconfig/network-scripts/ifcfg-ens33`,文件内容如下:
```bash
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp	# 这里表示ip为dhcp获取,静态ip为static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=0d519c59-8dfc-4eeb-917d-d0d51ab7d0f6
DEVICE=ens33
ONBOOT=no	# 这里表示开机加载网卡
```

- 配置静态ip，除了修改网卡为`static`之外，还需要为网卡配置IP地址和子网掩码以及网关，在虚拟机中网关可以在虚拟网络编辑器中看到：
![网关地址](http://p5qynomrl.bkt.clouddn.com/15215702249616hbmgtp3.png?imageslim)
- 具体修改**ens33**配置文件如下：
```bash
TYPE=Ethernet
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
NAME=ens33
UUID=0d519c59-8dfc-4eeb-917d-d0d51ab7d0f6
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.253.133	# 添加IP地址
NETMASK=255.255.255.0	# 配置子网掩码
GATEWAY=192.168.253.2	# 配置网关
DNS1=114.114.114.114	# 配置DNS
```

## 重启网络服务
- 在不重启的情况下让IP地址生效，需要重启网络服务，使用命令`systemctl restart network.service`：
```bash
[root@localhost ~]# systemctl restart network.service
[root@localhost ~]# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:71:8a:c0 brd ff:ff:ff:ff:ff:ff
    inet 192.168.253.133/24 brd 192.168.253.255 scope global ens33
       valid_lft forever preferred_lft forever
    inet6 fe80::b09c:fbff:1cfe:b7e7/64 scope link
       valid_lft forever preferred_lft forever
```

- 可以看到ens33的IP地址已经改变为在配置文件中配置的`192.168.253.133`。
- 检查网络连通性，使用`ping`命令：
```bash
[root@localhost ~]# ping www.evobot.cn
PING sni.github.map.fastly.net (151.101.73.147) 56(84) bytes of data.
64 bytes from 151.101.73.147 (151.101.73.147): icmp_seq=1 ttl=128 time=91.1 ms
64 bytes from 151.101.73.147 (151.101.73.147): icmp_seq=2 ttl=128 time=90.5 ms
64 bytes from 151.101.73.147 (151.101.73.147): icmp_seq=3 ttl=128 time=94.1 ms
^C
--- sni.github.map.fastly.net ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2004ms
rtt min/avg/max/mdev = 90.520/91.953/94.169/1.627 ms
```

# 网络问题排查
- 有时配置了IP地址的时候，网络可能会遇到不通的情况，这时可以重新配置为DHCP模式，重启dhclient服务,使用`dhclient -r`命令重新获取IP地址。
- 使用`route -n`命令查看系统路由网关是否存在：
```bash
[root@localhost ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.253.2   0.0.0.0         UG    100    0        0 ens33
192.168.253.0   0.0.0.0         255.255.255.0   U     100    0        0 ens33
```

- 如果在执行`ifconfig`命令时提示命令不存在，则需要安装`net-tools`软件包,使用命令`yum install -y net-tools`进行安装。

---



