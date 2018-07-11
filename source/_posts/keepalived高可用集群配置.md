---
title: keepalived高可用集群配置
author: Evobot
categories: keepalived
tags:
  - 高可用
  - Centos
  - keepalived
abbrlink: d985ce99
date: 2018-07-03 21:05:21
image:
---



本文主要对集群和keepalived进行了相关的介绍，然后介绍了如何进行keepalived高可用集群的配置和测试。

<!--more-->

---

# 集群介绍

- Linux集群按照功能划分可以分为两大类：高可用和负载均衡；
- 高可用集群介绍：
  - 高可用集群通常为两台服务器，一台工作，另外一台作为冗余，当提供服务的机器宕机，冗余将阶梯继续提供服务；
  - 实现高可用的开源软件主要有：heartbeat和keepalived；
- 负载均衡集群介绍：
  - 负载均衡集群需要有一台服务器作为分发器，负责将用户请求分发给后端的服务器处理，在这个集群里，除了分发器外，就是给用户提供服务的服务器，这些服务器的数量至少为2；
  - 实现负载均衡的开源软件有：LVS、keepalived、haproxy、nginx，商业软件则有F5、Netscaler。

---

# keepalived介绍

- keepalived通过VRRP(Virtual Router Redundancy Protocl)协议来实现搞可用，即虚拟路由冗余协议。
- VRRP协议中，会将多台功能相同的路由器组成一个小组，小组中会有一个master角色，和N(N>=1)个backup角色；
- master会通过组播形式向各个backup发送VRRP协议的数据包，当backup收不到master发来的VRRP数据包时，就会认为master宕机，此时会根据各个backup的优先级决定谁来成为新的master；
- keepalived有三个模块，分别为core、check和vrrp。其中core模块为keepalived的核心，负责主进程启动、维护以及全局配置文件的加载和解析，check模块负责健康检查，vrrp模块则是用来实现VRRP协议。

# 配置keepalived高可用集群

## 环境准备

- 首先准备两台机器，如ip为192.168.199.128的master和192.168.199.129的backup；

- 在两台机器上使用yum安装keepalived，并安装nginx，nginx的安装可以参考之前的博文：[Nginx 安装及默认虚拟主机、用户认证、域名重定向](https://www.evobot.cn/post/1785e4e8.html)：

  ```bash
  yum install -y keepalived
  ```

- 我们使用nginx作为高可用的对象，因为在生产环境中，nginx常被作为负载均衡器，所以一旦nginx出现单点故障，会导致后端的web服务器均不可被访问；

## 配置master的keepalived

- keepalived的配置文件是`/etc/keepalived/keepalived.conf`，我们将其重命名备份或清空内容，并新建一个keepalived.conf文件，写入以下内容，作为master的配置：

  ```bash
  #全局定义参数
  global_defs {
     #出现问题时向邮箱发送邮件
     notification_email {
       evobot@evobot.cn
     }
     # 邮件的发送地址，可以使用三方smtp服务
     notification_email_from root@evobot.cn
     smtp_server 127.0.0.1
     smtp_connect_timeout 30
     router_id LVS_DEVEL
  }

  # 检测服务是否正常，check模块执行的工作
  vrrp_script chk_nginx {
      # 检测服务的shell脚本路径
      script "/usr/local/sbin/check_ng.sh"
      # 检测间隔3秒
      interval 3
  }

  # 定义master相关配置
  vrrp_instance VI_1 {
      #定义角色master
      state MASTER
      #广播使用的网卡
      interface ens32
      #定义路由器ID
      virtual_router_id 51
      #定义master权重
      priority 100
      advert_int 1
      #认证相关信息
      authentication {
          #密码认证
          auth_type PASS
          #密码字符串
          auth_pass evobot>cn
      }
      # 定义master和backup公有的VIP(虚拟IP)
      virtual_ipaddress {
          192.168.199.127
      }
      #加载检查脚本
      track_script {
          chk_nginx
      }

  }

  ```

- 定义检测脚本check_ng.sh，文件路径为`/usr/local/sbin/`目录下，写入以下内容：

  ```bash
  #!/bin/bash
  #时间变量，用于记录日志
  d=`date --date today +%Y%m%d_%H:%M:%S`
  #计算nginx进程数量
  n=`ps -C nginx --no-heading|wc -l`
  #如果进程为0，则启动nginx，并且再次检测nginx进程数量，
  #如果还为0，说明nginx无法启动，此时需要关闭keepalived，以便backup启动服务
  if [ $n -eq "0" ]; then
          /etc/init.d/nginx start
          n2=`ps -C nginx --no-heading|wc -l`
          if [ $n2 -eq "0"  ]; then
                  echo "$d nginx down,keepalived will stop" >> /var/log/check_ng.log
                  systemctl stop keepalived
          fi
  fi


  ```

- 给check_ng.sh增加执行权限：

  ```bash
  chmod 755 /usr/local/sbin/check_ng.sh
  ```

- 然后启动master上的keepalived服务，在这里我们可以尝试先停止master上的nginx服务，然后启动keepalived服务，再查看keepalived和nginx进程状态：

  ```bash
  $ ps aux |grep nginx
  root       8402  0.0  0.2 112724   976 pts/1    R+   23:31   0:00 grep --color=auto nginx

  $ systemctl start keepalived

  $ ps aux |grep keep
  root       8410  0.0  0.2 118632  1392 ?        Ss   23:31   0:00 /usr/sbin/keepalived -D
  root       8411  0.0  0.5 118632  2608 ?        S    23:31   0:00 /usr/sbin/keepalived -D
  root       8412  0.0  0.5 120756  2516 ?        S    23:31   0:00 /usr/sbin/keepalived -D
  root       8445  0.0  0.2 112724   976 pts/1    S+   23:31   0:00 grep --color=auto keep

  $ ps aux |grep nginx
  root       8433  0.0  0.1  20548   608 ?        Ss   23:31   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
  nobody     8437  0.0  0.2  20988  1068 ?        S    23:31   0:00 nginx: worker process
  root       8453  0.0  0.2 112724   976 pts/1    R+   23:31   0:00 grep --color=auto nginx

  ```

- 在keepalived运行过程中，即使将nginx停止，keepalived也会自动重启nginx服务：

  ```bash
  $ service  nginx stop
  Stopping nginx (via systemctl):                            [  确定  ]

  $ !ps
  ps aux |grep nginx
  root       8781  0.0  0.1  20548   604 ?        Ss   23:34   0:00 nginx: master process /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
  nobody     8785  0.0  0.2  20988  1068 ?        S    23:34   0:00 nginx: worker process
  root       8787  0.0  0.2 112724   976 pts/1    R+   23:34   0:00 grep --color=auto nginx

  $ date
  2018年 07月 03日 星期二 23:34:10 CST

  ```

- keepalived的日志记录在`/var/log/messages`文件中：

  ```bash
  $ tail /var/log/messages
  Jul  3 23:31:48 128 Keepalived_vrrp[8412]: Sending gratuitous ARP on ens32 for 192.168.199.127
  Jul  3 23:31:48 128 Keepalived_vrrp[8412]: Sending gratuitous ARP on ens32 for 192.168.199.127
  Jul  3 23:31:48 128 Keepalived_vrrp[8412]: Sending gratuitous ARP on ens32 for 192.168.199.127
  Jul  3 23:31:48 128 Keepalived_vrrp[8412]: Sending gratuitous ARP on ens32 for 192.168.199.127
  Jul  3 23:34:05 128 systemd: Stopping SYSV: http service....
  Jul  3 23:34:05 128 nginx: Stopping Nginx: [  确定  ]
  Jul  3 23:34:05 128 systemd: Stopped SYSV: http service..
  Jul  3 23:34:06 128 systemd: Starting SYSV: http service....
  Jul  3 23:34:06 128 nginx: Starting Nginx: [  确定  ]
  Jul  3 23:34:06 128 systemd: Started SYSV: http service..


  ```

- 而绑定的公用IP地址，需要使用`ip add`命令查看：

  ```bash
  $ ip add
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host 
         valid_lft forever preferred_lft forever
  2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
      link/ether 00:0c:29:82:ee:84 brd ff:ff:ff:ff:ff:ff
      inet 192.168.199.128/24 brd 192.168.199.255 scope global ens32
         valid_lft forever preferred_lft forever
      inet 192.168.199.127/32 scope global ens32
         valid_lft forever preferred_lft forever
      inet6 fe80::c4b:209:685b:15da/64 scope link 
         valid_lft forever preferred_lft forever
      inet6 fe80::e2ac:d368:af2c:1e12/64 scope link tentative dadfailed 
         valid_lft forever preferred_lft forever

  ```

- 为了master和backup通信正常，需要关闭防火墙和selinux。


## 配置backup的keepalived

- 同样的，清空backup上的keepalived.conf文件，写入以下内容：

  ```bash
  global_defs {
     notification_email {
       evobot@evobot.cn
     }
     notification_email_from root@evobot.cn
     smtp_server 127.0.0.1
     smtp_connect_timeout 30
     router_id LVS_DEVEL
  }

  vrrp_script chk_nginx {
      script "/usr/local/sbin/check_ng.sh"
      interval 3
  }

  vrrp_instance VI_1 {
      #角色为backup
      state BACKUP
      interface ens32
      #虚拟路由id要与master一致
      virtual_router_id 51
      #权重要比master低
      priority 90
      advert_int 1
      authentication {
          auth_type PASS
          auth_pass evobot>cn
      }
      virtual_ipaddress {
          192.168.199.127
      }

      track_script {
          chk_nginx
      }

  }

  ```

- 创建nginx服务检测脚本check_ng.sh并修改权限，内容与master相同，如果backup上的nginx是yum安装，则启动nginx的命令需要自行修改；

- 然后启动keepalived服务，并查看相关进程：

  ```bash
  $ systemctl start keepalived

  $ ps aux |grep keep
  root       7737  0.0  0.2 118140  1384 ?        Ss   23:55   0:00 /usr/sbin/keepalived -D
  root       7738  0.0  0.5 118140  2604 ?        S    23:55   0:00 /usr/sbin/keepalived -D
  root       7739  0.0  0.4 120264  2360 ?        S    23:55   0:00 /usr/sbin/keepalived -D
  root       7753  0.0  0.2 112724   976 pts/0    R+   23:55   0:00 grep --color=auto keep

  $ ps aux |grep nginx
  root       7639  0.0  0.1  20548   608 ?        Ss   22:34   0:00 nginx: master process sbin/nginx
  nobody     7640  0.0  0.2  20992  1068 ?        S    22:34   0:00 nginx: worker process
  root       7767  0.0  0.2 112724   976 pts/0    R+   23:55   0:00 grep --color=auto nginx

  ```

## 测试keepalived服务

- 首先分别将两台机器的nginx主页修改，访问master的IP地址192.168.199.128，页面如下：

  ![master-server](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/master-server.png)

- 访问backup服务器的主页，页面如下：

  ![backup-server](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/backup-server.png)

- 我们为keepalived配置的公有IP是192.168.199.127，访问这个地址，显示的页面如下：

  ![keepalived-master](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/keepalived-master.png)

  - 说明目前的VIP是在master上面的。

- 我们尝试将master上的keepalived服务停止，然后重新访问：

  ```bash
  #在master上执行
  $ systemctl stop keepalived
  ```

  页面显示如下：

  ![keepalived-backup](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/keepalived-backup.png)


  - VIP已经转移到backup服务器上。

---

