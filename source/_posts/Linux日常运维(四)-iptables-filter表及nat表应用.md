---
title: Linux日常运维(四)—iptables filter表及nat表应用
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: f587388e
date: 2018-05-09 21:18:08
image:
---



本文继续介绍iptables的应用，主要针对filter表的一些案例进行讲解，以及nat表的各类应用方法进行介绍。

<!--more-->

---

# iptables小案例

- 需求：允许21、22、80端口的数据包进入，但22端口的只允许指定的IP地址段的数据包进入，使用shell脚本操作，脚本如下：

  ```bash
  #!/bin/bash
  ipt="/usr/sbin/iptables"
  $ipt -F
  $ipt -P INPUT DROP
  $ipt -P OUTPUT ACCEPT
  $ipt -P FORWARD ACCEPT
  # -m state指定tcp的状态，RELATED表示在连接完成后额外一些连接的状态，ESTABLISHED表示连接状态
  # 这条规则让通信更加顺畅
  $ipt -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
  $ipt -A INPUT -s 192.168.199.0/24 -p tcp --dport 22 -j ACCEPT
  $ipt -A INPUT -p tcp --dport 80 -j ACCEPT
  $ipt -A INPUT -p tcp --dport 21 -j ACCEPT
  ```

- 执行脚本后，可以看到规则已经生效：

  ```bash
  [root@www ~]# iptables -nvL
  Chain INPUT (policy DROP 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination         
     30  2072 ACCEPT     all  --  *      *       0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
      0     0 ACCEPT     tcp  --  *      *       192.168.199.0/24     0.0.0.0/0            tcp dpt:22
      0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80
      0     0 ACCEPT     tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            tcp dpt:21

  Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain OUTPUT (policy ACCEPT 21 packets, 2656 bytes)
   pkts bytes target     prot opt in     out     source               destination         
  ```

- 需求：禁止其他机器ping通本机；其实上面的shell脚本内的规则也会禁止ping，另外ping命令使用的是icmp协议，所以可以使用下面的规则，其中`--icmp-type 8`指的时icmp协议的ping请求类型：

  ```bash
  [root@www ~]# iptables -I INPUT -p icmp --icmp-type 8 -j DROP
  ```

- 放通ping包则使用下面的规则：

  ```bash
  [root@www ~]# iptables -I INPUT -p icmp --icmp-type 8 -j ACCEPT
  ```

- 关于icmp的type，可以参考[ICMP-type对应表](http://www.361way.com/icmp-type/1186.html)

# nat表应用

- 案例：A机器有两块网卡，分别是ens33(192.168.199.224)和ens37(192.168.100.2)，其中ens33可以上外网，ens77连接内部局域网，B机器则只有ens37(192.168.100.100)网卡，和A机器的ens37能够互通。

- 需求1：让B机器能够连接外网；从下图可以看到两台机器内网可通：

  ![ip addr](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/ipaddr.png)


  1. A机器打开路由转发功能，默认为0关闭状态：

     `echo 1 > /proc/sys/net/ipv4/ip_forward`

  2. A机器iptables添加规则：

     `iptables -t nat -A POSTROUTING -s 192.168.100.0/24 -o enp0s3 -j MASQUERADE`

  3. B机器设置网关为**192.168.100.2**，即A机器的内网地址：

     `route add default gw 192.168.100.2`

  4. 这时从B机器上已经能够ping通外网。


- 需求2：C机器能够和A通信，让C机器能够直接连接到B机器的22端口，即需要做端口映射：

  - 首先依然需要A上打开路由转发：`echo "1" > /proc/sys/net/ipv4/ip_forward`;

  - A上添加iptables规则将进来的数据包做目的地址转发DNAT到B机器IP上：

    `iptables -t nat -A PREROUTING -d 192.168.199.224 -p tcp --dport 1122 -j DNAT --to 192.168.100.100:22`

  - 然后A再添加规则将从B机器过来的数据包做源地址转发SNAT到A的IP上:

    `iptables -t nat -A POSTROUTING -s 192.168.100.100 -j SNAT --to 192.168.199.224`

  - B机器添加A机器内网地址为网关，然后就可以使用ssh链接192.168.199.224:1122即可到达B机器。

- 案例2：针对一个网段设置规则：

  ```bash
  $ iptables -I INPUT -m iprange --src-range 61.4.176.0-61.4.176.255 -j DROP
  ```

- 案例3：每5s内tcp3次握手超过20次的属于不正常访问，需要对这样的连接DROP处理： 

  ```bash
  $ iptables -A INPUT -s 192.168.0.0/255.255.255.0 -d 192.168.0.101 -p tcp -m tcp --dport 80 -m state --state NEW -m recent --set --name httpuser --rsource
  $ iptables -A INPUT -m recent --update --seconds 5 --hitcount 20 --name httpuser --rsource -j DROP
  ```
# iptables扩展

- iptables中的DNAT、SNAT和MASQUERADE分别叫做目的地址转换、源地址转换和动态源地址转换；
- 如下图所示的iptables的流程图：
  ![iptables flow](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/iptables_nat.png)
- 正菱形的区域是对数据包进行判定转发的地方。在这里，系统会根据IP数据包中的destination ip address中的IP地址对数据包进行分发。如果destination ip adress是本机地址，数据将会被转交给INPUT链。如果不是本机地址，则交给FORWARD链检测。
- DNAT要在进入这个菱形转发区域之前，也就是在PREROUTING链中做，比如我们要把访问202.103.96.112的访问转发到192.168.0.112上：

  ```bash
  iptables -t nat -A PREROUTING -d 202.103.96.112 -j DNAT --to-destination 192.168.0.112
  ```

  - 这个转换过程当中，其实就是将已经达到这台Linux网关（防火墙）上的数据包上的destination ip address从202.103.96.112修改为192.168.0.112然后交给系统路由进行转发。
  - 而SNAT自然是要在数据包流出这台机器之前的最后一个链也就是POSTROUTING链来进行操作:
  ```bash
  iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -j SNAT --to-source 58.20.51.66
  ```
  - 这个语句就是告诉系统把即将要流出本机的数据的source ip address修改成为58.20.51.66。这样，数据包在达到目的机器以后，目的机器会将包返回到58.20.51.66也就是本机。如果不做这个操作，那么你的数据包在传递的过程中，reply的包肯定会丢失。
  - 假如当前系统用的是ADSL/3G/4G动态拨号方式，那么每次拨号，出口IP都会改变，SNAT就会有局限性。
  ```bash
  iptables -t nat -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE
  ```
  - `MASQUERADE`这个设定值就是**IP伪装成为封包出去(-o)的那块装置上的IP**！不管现在eth0的出口获得了怎样的动态ip，MASQUERADE会自动读取eth0现在的ip地址然后做SNAT出去，这样就实现了很好的动态SNAT地址转换。

---
