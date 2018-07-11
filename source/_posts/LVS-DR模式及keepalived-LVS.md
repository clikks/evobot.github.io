---
title: LVS DR模式及keepalived+LVS
author: Evobot
categories: LVS
tags:
  - LVS
  - keepalived
  - 负载均衡
abbrlink: 92b2b429
date: 2018-07-05 21:34:23
image:
---



本文主要介绍LVS DR模式如何进行配置，另外介绍怎么使用keepalived实现LVS的负载均衡功能。

<!--more-->

---

# LVS DR模式搭建

## 环境准备

- 准备三台机器或虚拟机，分发器(dir)只需要一张网卡，IP设置为192.168.67.127,；
- rs1和rs2分别设置IP地址为192.168.67.129、192.168.67.130；
- 另外vip的地址为192.168.67.100。

##  配置DR模式

- 在dir上创建`/usr/local/sbin/lvs_dr.sh`shell脚本，内容如下，根据自己的配置更改：

  ```bash
  #! /bin/bash
  echo 1 > /proc/sys/net/ipv4/ip_forward
  ipv=/usr/sbin/ipvsadm
  vip=192.168.67.100
  rs1=192.168.67.129
  rs2=192.168.67.130
  ifdown ens32
  ifup ens32
  ifconfig ens32:2 $vip broadcast $vip netmask 255.255.255.255 up
  route add -host $vip dev ens32:2
  $ipv -C
  $ipv -A -t $vip:80 -s wrr
  # -g表示DR模式
  $ipv -a -t $vip:80 -r $rs1:80 -g -w 1
  $ipv -a -t $vip:80 -r $rs2:80 -g -w 1

  ```

- 然后在两台rs上同样创建脚本`/usr/local/sbin/lvs_rs.sh`，内容如下：

  ```bash
  #! /bin/bash
  vip=192.168.67.100
  #把vip绑定在lo上，是为了实现rs直接将结果返回给客户端
  ifdown lo
  ifup lo
  ifconfig lo:0 $vip broadcast $vip netmask 255.255.255.255 up
  route add -host $vip lo:0
  #下面是为了更改arp内核参数，目的是让rs顺利发送mac地址给客户端
  #参考文档www.cnblogs.com/lgfeng/archive/2012/10/16/2726308.html
  echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
  echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
  echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
  echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce

  ```

- 最后在dir以及两台rs上分别执行上面创建的脚本，然后尝试访问vip，注意，如果在浏览器中测试，刷新时需要ctrl+F5进行刷新，如果使用curl访问，则不能在dir或者rs上进行访问，要从其他linux上访问，结果如下：

  ```bash
  $ curl 192.168.67.100                            This is rs1 server!

  $ curl 192.168.67.100
  This is rs2 server!

  $ curl 192.168.67.100
  This is rs1 server!

  $ curl 192.168.67.100
  This is rs2 server!
  ```

  查看dir上：

  ```bash
  [root@load-balancer ~]# ipvsadm -ln
  IP Virtual Server version 1.2.1 (size=4096)
  Prot LocalAddress:Port Scheduler Flags
    -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
  TCP  192.168.67.100:80 wrr
    -> 192.168.67.129:80            Route   1      0          4         
    -> 192.168.67.130:80            Route   1      0          5         
  [root@load-balancer ~]# ipvsadm -ln
  IP Virtual Server version 1.2.1 (size=4096)
  Prot LocalAddress:Port Scheduler Flags
    -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
  TCP  192.168.67.100:80 wrr
    -> 192.168.67.129:80            Route   1      1          4         
    -> 192.168.67.130:80            Route   1      0          5         
  [root@load-balancer ~]# ipvsadm -ln
  IP Virtual Server version 1.2.1 (size=4096)
  Prot LocalAddress:Port Scheduler Flags
    -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
  TCP  192.168.67.100:80 wrr
    -> 192.168.67.129:80            Route   1      0          5         
    -> 192.168.67.130:80            Route   1      1          5         

  ```

  可以看到当前活跃的链接和不活跃的链接，并且两台rs都被分配到了请求。

---

# keepalived+LVS DR

## 环境准备

- 由于LVS中的dir很容易造成单点故障，并且LVS在rs出现宕机时，不能够自动的剔除宕机的rs，从而请求依旧被发送到宕机的rs上，造成访问出现问题，所以如果能够实现dir的高可用，那么整个服务会更强壮，而keepalived又内置了ipvsadm，并且又能够实现高可用，所以只需要两台keepalived服务器就可以实现高可用的负载均衡；
- 前面已经对keepalived的高可用进行过演示，这里主要演示keepalived的负载均衡功能，所以只配置一台keepalived服务器，其ip配置为192.168.67.127；
- 两台rs的地址分别为192.168.67.129和192.168.67.130；
- vip则为192.168.67.100。

## keepalived配置

- 编辑keepalived配置文件`/etc/keepalived/keepalived.conf`，内容如下：

  ```bash
  vrrp_instance VI_1 {
      #备用服务器上为 BACKUP
      state MASTER
      #绑定vip的网卡为ens32
      interface ens32
      virtual_router_id 51
      #备用服务器上为90
      priority 100
      advert_int 1
      authentication {
          auth_type PASS
          auth_pass evobot
      }
      virtual_ipaddress {
          192.168.67.100
      }
  }
  virtual_server 192.168.67.100 80 {
      #(每隔10秒查询realserver状态)
      delay_loop 10
      #(lvs 算法)
      lb_algo wlc
      #(DR模式)
      lb_kind DR
      #(同一IP的连接60秒内被分配到同一台realserver)
      persistence_timeout 60
      #(用TCP协议检查realserver状态)
      protocol TCP

      real_server 192.168.67.129 80 {
          #(权重)
          weight 100
          TCP_CHECK {
          #(10秒无响应超时)
          connect_timeout 10
          nb_get_retry 3
          delay_before_retry 3
          connect_port 80
          }
      }
      real_server 192.168.67.130 80 {
          weight 100
          TCP_CHECK {
          connect_timeout 10
          nb_get_retry 3
          delay_before_retry 3
          connect_port 80
          }
       }
  }

  ```

- 然后启动keepalived，注意启动前需要将之前LVS的配置清空，需要注意的是，虽然dir上使用keepalived实现负载均衡，但是在两台rs上，**依旧需要执行之前配置LVS时执行的shell脚本。**

- 使用keepalived实现的负载均衡，会在rs服务宕机时自动将rs剔除，如将一台rs上的nginx服务关闭，等待10s，查看ipvsadm规则如下，宕机的rs被剔除：

  ```bash
  [root@load-balancer ~]# ipvsadm -ln
  IP Virtual Server version 1.2.1 (size=4096)
  Prot LocalAddress:Port Scheduler Flags
    -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
  TCP  192.168.67.100:80 wlc persistent 60
    -> 192.168.67.129:80            Route   100    0          0         
    -> 192.168.67.130:80            Route   100    0          0        
    
  [root@load-balancer ~]# ipvsadm -ln
  IP Virtual Server version 1.2.1 (size=4096)
  Prot LocalAddress:Port Scheduler Flags
    -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
  TCP  192.168.67.100:80 wlc persistent 60
    -> 192.168.67.130:80            Route   100    0          0         

  ```

---