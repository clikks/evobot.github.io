---
title: redis集群
author: Evobot
date: 2018-08-22 22:13:01
categories: NoSQL
tags:
  - redis
image:
---

1. redis集群介绍 
2.  redis集群搭建配置 
3.  redis集群操作 

<!--more-->

---

# redis集群介绍

- redis集群官方称为redis-cluster，是在redis3.0版本之后开始支持的；
- redis集群可以实现多个redis节点网络互联、数据共享；
- 所有的节点都是一主一从或一主多从，其中从不提供服务，仅作为备用；
- redis集群不支持同时处理多个键（mset/mget），因为redis需要把键均为分布在各个节点上，并发量高的情况下，同时创建键值会降低性能并导致不可预测的行为；
- 支持在线增加、删除节点；
- 客户端可以连接任何一个主节点进行读写。

# redis集群配置

## 集群架构

- redis集群需要ruby2.2以上版本，并且只需要一台机器运行；

- 架构设置：
  1. 两台机器，分别开启三个redis服务；
  2. A机器上三个端口7000，7002，7004，全部为主
  3. B机器上三个端口7001，7003，7005，全部为从
  4. 两台机器上都要编译安装redis，然后编辑并复制3个不通的redis.conf配置文件，分别设置不同的端口号、dir等参数，还需要增加cluster相关的参数，然后启动6个redis服务,cluster参数如下：

  ```bash
  cluster-enabled yes	#开启集群
  cluster-config-file nodes-6379.conf  #不同的节点文件名也要不同，该文件会在dir配置的目录中自动生成
  cluster-node-timeout 10100
  ```

  5. 6个redis服务的配置如下:

  ```bash
  port 7000
  bind 192.168.133.130
  daemonize yes
  pidfile /var/run/redis_7000.pid
  dir /data/redis_data/7000
  cluster-enabled yes
  cluster-config-file nodes_7000.conf
  cluster-node-timeout 10100
  appendonly yes
  ```

  6. 启动redis服务后，进程如下：

  ```bash
  $ ps aux |grep redis
  root       7909  0.1  1.2 147356  2784 ?        Ssl  01:50   0:00 redis-server 192.168.78.131:7001 [cluster]
  root       7914  0.1  1.2 147356  2780 ?        Ssl  01:51   0:00 redis-server 192.168.78.131:7003 [cluster]
  root       7919  0.1  1.2 147356  2780 ?        Ssl  01:51   0:00 redis-server 192.168.78.131:7005 [cluster]
  
  ```

  

## 配置集群

1.  安装ruby：

   - 方法1：

   ```bash
   $ yum groupinstall -y "Development Tools"
   $ yum install -y gdbm-devel libdb4-devel libffi-devel libyaml libyaml-devel ncurses-devel openssl-devel readline-devel tcl-devel
   $ cd /root/
   $ mkdir -p rpmbuild/{BUILD,BUILDROOT,RPMS,SOURCES,SPECS,SRPMS}
   # 下载ruby源码包
   $ wget https://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.3.tar.gz -p rpmbuild/SOURCES/
   $ wget https://raw.githubusercontent.com/tjinjin/automate-ruby-rpm/master/ruby22x.spec -P rpmbuild/SPECS
   $ rpmbuild -bb rpmbuild/SPECS/ruby22x.spec
   # 使用yum安装本地rpm软件包
   $ yum -y localinstall rpmbuild/RPMS/x86_64/ruby-2.2.3-1.el7.centos.x86_64.rpm
   $ gem install redis
   ```

   - 方法2：

   访问[rvm.io](http://rvm.io/)，官方提供的安装ruby的方法：

   ```bash
   $ gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
   
   $ curl -sSL https://get.rvm.io | bash -s stable
   $ source /etc/profile.d/rvm.sh
   $ rvm list known # 查看可以安装的ruby版本
   $ rvm install 2.2.10 # 安装所需的版本
   $ gem install redis
   ```

2. redis-trib.rb命令：

   ```bash
   $ cp /usr/local/src/redis-4.0.11/src/redis-trib.rb /usr/bin/
   $ redis-trib.rb create --replicas 1 192.168.78.130:7000 192.168.78.130:7002 192.168.78.130:7004 192.168.78.131:7001 192.168.78.131:7003 192.168.78.131:7005
   >>> Creating cluster
   >>> Performing hash slots allocation on 6 nodes...
   Using 3 masters:	# 自动分配master
   192.168.78.130:7000
   192.168.78.131:7001
   192.168.78.130:7002
   # 自动分配slave
   Adding replica 192.168.78.131:7005 to 192.168.78.130:7000
   Adding replica 192.168.78.130:7004 to 192.168.78.131:7001
   Adding replica 192.168.78.131:7003 to 192.168.78.130:7002
   M: aac6fd26e2ec4d8d261a6def8e634d13008fe82c 192.168.78.130:7000
      slots:0-5460 (5461 slots) master
   M: 4b9454aa3c7ffbc88e9e917213a13814ee5488fd 192.168.78.130:7002
      slots:10923-16383 (5461 slots) master
   S: 8bd6d130b50854224bcc016193053c14b364c18c 192.168.78.130:7004
      replicates 901ffea2f83e552e6991711dc3e39d81116f3f61
   M: 901ffea2f83e552e6991711dc3e39d81116f3f61 192.168.78.131:7001
      slots:5461-10922 (5462 slots) master
   S: 14f4947795e4c7b2774ae11bd4692a837eb98f09 192.168.78.131:7003
      replicates 4b9454aa3c7ffbc88e9e917213a13814ee5488fd
   S: 93a1dc5d65820876edfa5549872f35b964ca3bbd 192.168.78.131:7005
      replicates aac6fd26e2ec4d8d261a6def8e634d13008fe82c
   Can I set the above configuration? (type 'yes' to accept):yes
   >>> Nodes configuration updated
   >>> Assign a different config epoch to each node
   >>> Sending CLUSTER MEET messages to join the cluster
   Waiting for the cluster to join.....
   >>> Performing Cluster Check (using node 192.168.78.130:7000)
   M: aac6fd26e2ec4d8d261a6def8e634d13008fe82c 192.168.78.130:7000
      slots:0-5460 (5461 slots) master
      1 additional replica(s)
   S: 93a1dc5d65820876edfa5549872f35b964ca3bbd 192.168.78.131:7005
      slots: (0 slots) slave
      replicates aac6fd26e2ec4d8d261a6def8e634d13008fe82c
   M: 901ffea2f83e552e6991711dc3e39d81116f3f61 192.168.78.131:7001
      slots:5461-10922 (5462 slots) master
      1 additional replica(s)
   S: 8bd6d130b50854224bcc016193053c14b364c18c 192.168.78.130:7004
      slots: (0 slots) slave
      replicates 901ffea2f83e552e6991711dc3e39d81116f3f61
   S: 14f4947795e4c7b2774ae11bd4692a837eb98f09 192.168.78.131:7003
      slots: (0 slots) slave
      replicates 4b9454aa3c7ffbc88e9e917213a13814ee5488fd
   M: 4b9454aa3c7ffbc88e9e917213a13814ee5488fd 192.168.78.130:7002
      slots:10923-16383 (5461 slots) master
      1 additional replica(s)
   [OK] All nodes agree about slots configuration.
   >>> Check for open slots...
   >>> Check slots coverage...
   [OK] All 16384 slots covered.
   
   ```

# 集群操作

- redis集群登陆命令行可以使用节点中的任意一个端口，使用`-c`选项以集群方式登陆，`-h`指定host：

  ```bash
  $ redis-cli -c -h 192.168.78.130 -p 7000
  ```

- 任意一个节点都可以创建或查看key，并且redis会自动分配存储key的节点，并且会打印相应的消息：

  ```bash
  192.168.78.130:7000> set key1 aaa
  -> Redirected to slot [9189] located at 192.168.78.131:7001
  OK
  192.168.78.131:7001> set key2 bbb
  -> Redirected to slot [4998] located at 192.168.78.130:7000
  OK
  # 没有打印保存到哪个节点就表示存在本机
  192.168.78.130:7000> set key3 ccc
  OK
  
  ```

- 同样的查看key也会显示相关信息：

  ```bash
  192.168.78.130:7002> get key1
  -> Redirected to slot [9189] located at 192.168.78.131:7001
  "aaa"
  192.168.78.131:7001> get key2
  -> Redirected to slot [4998] located at 192.168.78.130:7000
  "bbb"
  192.168.78.130:7000> get key3
  "eee"
  
  ```

- 检测集群状态，使用`redis-trib.rb check [ip:port]`命令，其中ip和port可以使用集群中任意一个节点：

  ```bash
  $ redis-trib.rb check 192.168.78.130:7000
  >>> Performing Cluster Check (using node 192.168.78.130:7000)
  M: aac6fd26e2ec4d8d261a6def8e634d13008fe82c 192.168.78.130:7000
     slots:0-5460 (5461 slots) master
     1 additional replica(s)
  S: 93a1dc5d65820876edfa5549872f35b964ca3bbd 192.168.78.131:7005
     slots: (0 slots) slave
     replicates aac6fd26e2ec4d8d261a6def8e634d13008fe82c
  M: 901ffea2f83e552e6991711dc3e39d81116f3f61 192.168.78.131:7001
     slots:5461-10922 (5462 slots) master
     1 additional replica(s)
  S: 8bd6d130b50854224bcc016193053c14b364c18c 192.168.78.130:7004
     slots: (0 slots) slave
     replicates 901ffea2f83e552e6991711dc3e39d81116f3f61
  S: 14f4947795e4c7b2774ae11bd4692a837eb98f09 192.168.78.131:7003
     slots: (0 slots) slave
     replicates 4b9454aa3c7ffbc88e9e917213a13814ee5488fd
  M: 4b9454aa3c7ffbc88e9e917213a13814ee5488fd 192.168.78.130:7002
     slots:10923-16383 (5461 slots) master
     1 additional replica(s)
  [OK] All nodes agree about slots configuration.
  >>> Check for open slots...
  >>> Check slots coverage...
  [OK] All 16384 slots covered.
  
  ```

## redis集群管理命令

- `CLUSTER NODES`：查看集群节点：

  ```bash
  192.168.78.130:7000> CLUSTER NODES
  93a1dc5d65820876edfa5549872f35b964ca3bbd 192.168.78.131:7005@17005 slave aac6fd26e2ec4d8d261a6def8e634d13008fe82c 0 1535048218000 6 connected
  aac6fd26e2ec4d8d261a6def8e634d13008fe82c 192.168.78.130:7000@17000 myself,master - 0 1535048220000 1 connected 0-5460
  901ffea2f83e552e6991711dc3e39d81116f3f61 192.168.78.131:7001@17001 master - 0 1535048219723 4 connected 5461-10922
  8bd6d130b50854224bcc016193053c14b364c18c 192.168.78.130:7004@17004 slave 901ffea2f83e552e6991711dc3e39d81116f3f61 0 1535048219000 4 connected
  14f4947795e4c7b2774ae11bd4692a837eb98f09 192.168.78.131:7003@17003 slave 4b9454aa3c7ffbc88e9e917213a13814ee5488fd 0 1535048219000 5 connected
  4b9454aa3c7ffbc88e9e917213a13814ee5488fd 192.168.78.130:7002@17002 master - 0 1535048220730 2 connected 10923-16383
  
  ```

  该命令会标注节点是master还是slave，并且会用一串字符串表示slave对应的master，myself表示本机。

- `CLUSTER INFO`：查看集群信息：

  ```bash
  192.168.78.130:7000> CLUSTER INFO
  cluster_state:ok
  cluster_slots_assigned:16384
  cluster_slots_ok:16384
  cluster_slots_pfail:0
  cluster_slots_fail:0
  cluster_known_nodes:6
  cluster_size:3
  cluster_current_epoch:6
  cluster_my_epoch:1
  cluster_stats_messages_ping_sent:960
  cluster_stats_messages_pong_sent:1015
  cluster_stats_messages_sent:1975
  cluster_stats_messages_ping_received:1010
  cluster_stats_messages_pong_received:960
  cluster_stats_messages_meet_received:5
  cluster_stats_messages_received:1975
  
  ```

- `CLUSTER MEET [ip] [port]`：添加新节点，默认添加的新节点都是master节点：

  ```bash
  $ redis-cli -c -h 192.168.78.130 -p 7000
  192.168.78.130:7000> CLUSTER MEET 192.168.78.130 7006
  OK
  192.168.78.130:7000> cluster nodes
  93a1dc5d65820876edfa5549872f35b964ca3bbd 192.168.78.131:7005@17005 slave aac6fd26e2ec4d8d261a6def8e634d13008fe82c 0 1535049063785 6 connected
  # 默认添加为master节点
  7759f139d2b7ca0b4907bc75f101aa8be02d4e4d 192.168.78.130:7006@17006 master - 0 1535049064793 0 connected	
  aac6fd26e2ec4d8d261a6def8e634d13008fe82c 192.168.78.130:7000@17000 myself,master - 0 1535049063000 1 connected 0-5460
  901ffea2f83e552e6991711dc3e39d81116f3f61 192.168.78.131:7001@17001 master - 0 1535049062779 4 connected 5461-10922
  8bd6d130b50854224bcc016193053c14b364c18c 192.168.78.130:7004@17004 slave 901ffea2f83e552e6991711dc3e39d81116f3f61 0 1535049063000 4 connected
  14f4947795e4c7b2774ae11bd4692a837eb98f09 192.168.78.131:7003@17003 slave 4b9454aa3c7ffbc88e9e917213a13814ee5488fd 0 1535049062000 5 connected
  4b9454aa3c7ffbc88e9e917213a13814ee5488fd 192.168.78.130:7002@17002 master - 0 1535049063000 2 connected 10923-16383
  
  192.168.78.130:7000> CLUSTER MEET 192.168.78.131 7007
  OK
  192.168.78.130:7000> cluster nodes
  93a1dc5d65820876edfa5549872f35b964ca3bbd 192.168.78.131:7005@17005 slave aac6fd26e2ec4d8d261a6def8e634d13008fe82c 0 1535049121000 6 connected
  7759f139d2b7ca0b4907bc75f101aa8be02d4e4d 192.168.78.130:7006@17006 master - 0 1535049121635 0 connected
  aac6fd26e2ec4d8d261a6def8e634d13008fe82c 192.168.78.130:7000@17000 myself,master - 0 1535049119000 1 connected 0-5460
  901ffea2f83e552e6991711dc3e39d81116f3f61 192.168.78.131:7001@17001 master - 0 1535049121000 4 connected 5461-10922
  8bd6d130b50854224bcc016193053c14b364c18c 192.168.78.130:7004@17004 slave 901ffea2f83e552e6991711dc3e39d81116f3f61 0 1535049120121 4 connected
  14f4947795e4c7b2774ae11bd4692a837eb98f09 192.168.78.131:7003@17003 slave 4b9454aa3c7ffbc88e9e917213a13814ee5488fd 0 1535049122000 5 connected
  4b9454aa3c7ffbc88e9e917213a13814ee5488fd 192.168.78.130:7002@17002 master - 0 1535049122642 2 connected 10923-16383
  96f1283eb326bd6c3c7e67e0d8e715a56a5e749a 192.168.78.131:7007@17007 master - 0 1535049121334 0 connected
  
  ```

- `CLUSTER REPLICATE [node_id]`将当前节点设置为指定节点的从，需要先登陆到需要设置的节点上，例如前面增加的两个新节点，将7007设置为7006的从节点，首先登陆到7007：

  ```bash
  $ redis-cli -c -h 192.168.78.131 -p 7007
  
  # 复制7006的节点号
  192.168.78.131:7007> CLUSTER REPLICATE 7759f139d2b7ca0b4907bc75f101aa8be02d4e4d
  OK
  192.168.78.131:7007> CLUSTER NODES
  93a1dc5d65820876edfa5549872f35b964ca3bbd 192.168.78.131:7005@17005 slave aac6fd26e2ec4d8d261a6def8e634d13008fe82c 0 1535049357000 1 connected
  7759f139d2b7ca0b4907bc75f101aa8be02d4e4d 192.168.78.130:7006@17006 master - 0 1535049359774 7 connected
  14f4947795e4c7b2774ae11bd4692a837eb98f09 192.168.78.131:7003@17003 slave 4b9454aa3c7ffbc88e9e917213a13814ee5488fd 0 1535049357000 2 connected
  4b9454aa3c7ffbc88e9e917213a13814ee5488fd 192.168.78.130:7002@17002 master - 0 1535049357759 2 connected 10923-16383
  96f1283eb326bd6c3c7e67e0d8e715a56a5e749a 192.168.78.131:7007@17007 myself,slave 7759f139d2b7ca0b4907bc75f101aa8be02d4e4d 0 1535049356000 0 connected
  8bd6d130b50854224bcc016193053c14b364c18c 192.168.78.130:7004@17004 slave 901ffea2f83e552e6991711dc3e39d81116f3f61 0 1535049358767 4 connected
  aac6fd26e2ec4d8d261a6def8e634d13008fe82c 192.168.78.130:7000@17000 master - 0 1535049358000 1 connected 0-5460
  901ffea2f83e552e6991711dc3e39d81116f3f61 192.168.78.131:7001@17001 master - 0 1535049358000 4 connected 5461-10922
  
  ```

- `CLUSTER FORGET [node_id]`：移除一个从节点，注意要移除的节点不能为master，如果想要移除master，可以将其设置为另一个节点的从节点之后再进行删除作业：

  ```bash
  192.168.78.131:7007> CLUSTER FORGET 7759f139d2b7ca0b4907bc75f101aa8be02d4e4d
  (error) ERR Can't forget my master!
  
  # 也不能移除当前节点
  192.168.78.131:7007> CLUSTER FORGET 96f1283eb326bd6c3c7e67e0d8e715a56a5e749a
  (error) ERR I tried hard but I can't forget myself...
  
  $ redis-cli -c -h 192.168.78.130 -p 7000
  192.168.78.130:7000> CLUSTER FORGET 96f1283eb326bd6c3c7e67e0d8e715a56a5e749a
  OK
  
  ```

- `CLUSTER SAVECONFIG`：保存集群配置文件，会保存到cluster-config-file配置项指定的文件中：

  ```bash
  $ ls -l /data/redis7000/
  总用量 12
  -rw-r--r--. 1 root root 151 8月  24 02:11 appendonly.aof
  -rw-r--r--. 1 root root 202 8月  24 02:20 dump.rdb
  -rw-r--r--. 1 root root 915 8月  24 02:44 nodes-7000.conf
  $ redis-cli -c -h 192.168.78.130 -p 7000
  192.168.78.130:7000> CLUSTER SAVECONFIG
  OK
  192.168.78.130:7000>
  $ ls -l /data/redis7000/
  总用量 12
  -rw-r--r--. 1 root root 151 8月  24 02:11 appendonly.aof
  -rw-r--r--. 1 root root 202 8月  24 02:20 dump.rdb
  -rw-r--r--. 1 root root 915 8月  24 02:45 nodes-7000.conf
  
  ```

---

  

