---
title: LVS负载均衡集群配置
author: Evobot
categories: LVS
tags:
  - 负载均衡
  - LVS
  - Centos
abbrlink: 3e6b55c7
date: 2018-07-04 21:40:07
image:
---


本文主要对负载均衡集群和LVS进行介绍，以及LVS的三种运行模式，并演示了如何配置LVS NAT模式的负载均衡。


<!--more-->

---

# 负载均衡集群

## 负载均衡集群介绍

- 主流开源的负载均衡软件有LVS，keepalived、haproxy、bginx风
- LVS数据4层，nginx则属于7层(网络OSI7层模型)，haproxy既可以为4层，也可以当做7层使用；
- keepalived的负载均衡功能就是lvs，lvs这种4层的负载可以分发80端口以外的其他通信端口，比如MySQL；而nginx只可以支持http、https、mail，haproxy也可以支持MySQL；
- 相对来说，LVS这种4层结构更稳定，能承受更多的请求，而nginx这种7层的更改灵活，更容易实现个性化需求。

## LVS介绍

- LVS由国人章文嵩开发，流行度不亚于Apache的httpd，是基于TCP/IP做的路由和转发，稳定性和效率很高；
- LVS最新版本基于Linux内核2.6，并且很长时间没有再更新；
- LVS有三种常见的模式：NAT、DR、IP Tunnel；
- 在LVS架构中有一个核心角色叫做分发器(Load Balancer)，它用来分发用户的请求，还有诸多处理用户请求的服务器(Real Server简称rs)。

### **NAT模式**

- NAT模式是借助iptables的nat表来实现的；

- 用户的请求到分发器后，通过预设的iptables规则，将请求的数据包转发到后端的rs上去；

- rs需要设定网关为分发器的内网ip；

- 用户请求的数据包和返回给用户的数据包全部经过分发器，所以分发器成为瓶颈，所以一般NAT形式使用在后端服务器不多的情况下，如只处理后端十台服务器的服务；

- 在nat模式中，只需要分发器有公网IP即可，比较节省公网IP资源，架构图如下：

  ![LVS-NAT](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/LVS-NAT.png)

###  **IP Tunnel模式**

- 在这种模式中，需要有一个公共的IP配置在分发器和所有rs上，我们称之为vip；

- 客户端请求的目标IP为vip，分发器接收到请求数据包后，会对数据包做一个加工，会将目标IP改为rs的IP，这样数据包就到了rs上；

- rs接收数据包后，会还原原始数据包，这样目标IP为vip，因为所有rs上配置了这个vip，所以它会认为是它自己;

- rs处理完请求后就会使用vip直接将结果返回给客户端，不再经过分发器，这样分发器就不再是瓶颈，架构图如下：

  ![LVS-TUNNEL](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/LVS-TUNNEL.png)

###  **DR模式**

- 这种模式下，也需要一个公共的IP配置在分发器和所有rs上，即vip；

- 和IP Tunnel不同的是，它会把数据包的MAC地址修改为rs的MAC地址；

- rs接收数据包后，会还原原始数据包，这样目标IP为vip，因为所有rs上配置了这个vip，rs就会认为数据包是发送给自己的；

- rs处理完请求后，会通过vip直接将结果返回给客户端，架构图如下：

  ![LVS-DR](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/LVS-DR.png)

## LVS调度算法

1. 轮询 Round-Robin 简称rr，即请求会均衡的分发到每个rs上；
2. 加权轮询 Weight Round-Robin 简称wrr，相比1多了权重，权重高的分配到的请求数会多一些，如80比60的权重得到的请求多；
3. 最小连接 Least-Connection 简称lc，会将请求发送到请求量少rs上去；
4. 加权最小连接 Weight Least-Connection 简称wlc，相比3，多了权重；
5. 基于局部性的最小连接 Locality-Based Least  Connections 简称lblc；
6. 带复制的基于局部性最小连接 Locality-Based Least Connections with Replication 简称lblcr；
7. 目标地址散列调度 Destination Hashing 简称dh；
8. 源地址散列调度 Source Hashing 简称sh；

---

# LVS NAT模式搭建

## 环境准备

- 准备三台机器或虚拟机，其中一台为分发器，另外两台作为rs；

- 分发器，即调度器，简写为dir，具有两张网卡，配置LAN IP为172.16.126.128，外网ip为192.168.199.127，其中LAN IP都在虚拟机中使用host-only模式；

- rs1，LAN IP为172.16.126.129，网关设置为172.16.126.128；

- rs2，LAN IP为172.16.126.130，网关为172.16.126.128；

- 三台机器都需要关闭firewalld服务和selinux服务，并启动iptables服务，执行以下命令：

  ```bash
  $ systemctl stop firewalld

  $ systemctl disable firewalld

  $ yum install -y iptables-service

  $ systemctl start iptables

  $ systemctl ebable iptables

  $ iptables -F

  $ service iptables save
  ```

## NAT搭建

- 在dir上安装`ipvsadm`软件包，ipvsadm是实现LVS NAT模式的核心软件包：

  ```bash
  $ yum install -y ipvsadm
  ```

- 在dir上编写shell脚本，内容如下：

  ```bash
  $ vim /usr/local/sbin/lvs_nat.sh
  ```

  ```bash
  #!/bin/bash
  # director 服务器上开启路由转发功能
  echo 1 > /proc/sys/net/ipv4/ip_forward
  # 关闭icmp重定向
  echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects
  echo 0 > /proc/sys/net/ipv4/conf/default/send_redirects
  # 注意区分网卡名
  echo 0 > /proc/sys/net/ipv4/conf/ens32/send_redirects
  echo 0 > /proc/sys/net/ipv4/conf/ens35/send_redirects
  # director 设置nat防火墙
  iptables -t nat -F
  iptables -t nat -X
  iptables -t nat -A POSTROUTING -s 172.16.126.0/24 -j MASQUERADE
  # director设置ipvsadm
  IPVSADM='/usr/sbin/ipvsadm'
  # 清空规则
  $IPVSADM -C
  # -s指定算法，-p指定超时时间，单位秒，是请求在时间内请求到同一个rs，解决session问题
  $IPVSADM -A -t 192.169.199.127:80 -s lc -p 3
  # -r指定rs，-m指定nat模式，-w表示权重
  $IPVSADM -a -t 192.168.199.127:80 -r 172.16.126.129:80 -m -w 1
  $IPVSADM -a -t 192.168.199.127:80 -r 172.16.126.130:80 -m -w 1
  ```

  **上面的脚本最后两行命令执行时报错`Memory allocation problem`，所以改成了下面这种形式执行：**

  ```bash
  # !/bin/bash
  # director 服务器上开启路由转发功能
  echo 1 > /proc/sys/net/ipv4/ip_forward
  # 关闭icmp重定向
  echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects
  echo 0 > /proc/sys/net/ipv4/conf/default/send_redirects
  # 注意区分网卡名
  echo 0 > /proc/sys/net/ipv4/conf/ens32/send_redirects
  echo 0 > /proc/sys/net/ipv4/conf/ens35/send_redirects
  # director 设置nat防火墙
  iptables -t nat -F
  iptables -t nat -X
  iptables -t nat -A POSTROUTING -s 172.16.126.0/24 -j MASQUERADE
  # director设置ipvsadm
  IPVSADM='/usr/sbin/ipvsadm'
  $IPVSADM -C
  #$IPVSADM -A -t 192.169.199.127:80 -s lc -p 3
  #$IPVSADM -a -t 192.168.199.127:80 -r 172.16.126.129:80 -m -w 1
  #$IPVSADM -a -t 192.168.199.127:80 -r 172.16.126.130:80 -m -w 1
  echo "
  -A -t 192.168.199.127:80 -s lc -p 3
  -a -t 192.168.199.127:80 -r 172.16.126.129:80 -m -w 1
  -a -t 192.168.199.127:80 -r 172.16.126.130:80 -m -w 1
  "|ipvsadm -R

  ```

- 执行完成后，可以使用`iptables -t nat -nvL`查看iptables的nat表，使用`ipvsadm -ln`可以查看LVS的规则：

  ```bash
  [root@load-balancer sbin]# iptables -t nat -nvL
  Chain PREROUTING (policy ACCEPT 11 packets, 3007 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain INPUT (policy ACCEPT 5 packets, 2551 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain OUTPUT (policy ACCEPT 3 packets, 228 bytes)
   pkts bytes target     prot opt in     out     source               destination         

  Chain POSTROUTING (policy ACCEPT 3 packets, 228 bytes)
   pkts bytes target     prot opt in     out     source               destination         
      6   456 MASQUERADE  all  --  *      *       172.16.126.0/24      0.0.0.0/0           
      
  [root@load-balancer sbin]# ipvsadm -ln
  IP Virtual Server version 1.2.1 (size=4096)
  Prot LocalAddress:Port Scheduler Flags
    -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
  TCP  192.168.199.127:80 lc persistent 3
    -> 172.16.126.129:80            Masq    1      0          0         
    -> 172.16.126.130:80            Masq    1      0          0         

  ```

- 在rs1和rs2上启动nginx，并且对html页面进行修改，再使用curl访问dir的公网地址：

  ```bash
  [root@load-balancer sbin]# curl 192.168.199.127
  This is rs2 server!
  [root@load-balancer sbin]# curl 192.168.199.127
  This is rs1 server!
  [root@load-balancer sbin]# curl 192.168.199.127
  This is rs2 server!
  [root@load-balancer sbin]# curl 192.168.199.127
  This is rs1 server!
  [root@load-balancer sbin]# curl 192.168.199.127
  This is rs2 server!
  ```

  - 可以看到，客户端的请求会被调度到两台rs上，并不是每次都访问的同一台rs。

---

