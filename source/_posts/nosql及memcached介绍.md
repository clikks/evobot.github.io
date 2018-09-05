---
title: nosql及memcached介绍
author: Evobot
categories: NoSQL
tags:
  - NoSQL
  - memcached
abbrlink: ee0f8529
date: 2018-08-14 23:24:32
image:
---

1. nosql介绍
2. memrcached介绍
3. 安装memcached
4. 查看memcachedq状态

<!--more-->

---

# NoSQL介绍

## NoSQL的特点

- NoSQL即非关系型数据库，关系型数据库的代表是MySQL；
- 对于关系型数据库来说，需要将数据存储到库、表、行、字段里，查询的时候根据条件一行一行的进行匹配，当数据量非常大的时候就会很耗费时间和资源，尤其数据需要从磁盘里检索的时候；
- NoSQL数据库存储原理非常简单（典型的数据类型为k-v键值对），不存在复杂的关系链，比如MySQL在查询的时候，需要找到对应的库、表以及字段；
- NoSQL数据可以存储在内存里，查询速度非常快，虽然NoSQL在性能表现上虽然优于关系型数据库，但它并不能完全替代关系型数据库；
- NoSQL因为没有复杂的数据结构，扩展非常容易，并且支持分布式。

## 常见NoSQL数据库

- k-v形式的：memcached、redis，适合存储用户信息，如会话、配置文件、参数、购物车等等，这些信息一般都和ID(键)挂钩，这种情景下键值数据库是很好的选择；
- 文档数据库：mongodb，将数据以文档的形式存储，即JSON，每个文档都是一系列数据项的集合，每个数据项都有一个名称与对应的值，值既可以是简单的数据类型，如字符串、数字和日期等，也可以是复杂的类型，如有序列表和关联对象。数据存储的最小单位是文档，同一个表中存储的文档属性可以是不同的，数据可以使用XML、JSON或JSONB等多种形式存储；
- 列存储：Hbase；
- 图：Neo4J、infinite Graph、OrientDB。

# Memcached介绍

## Memcached简介

- Memcached是国外社区网站LiveJournal团队开发，目的是为了通过缓存数据库查询结果，减少数据库访问次数，从而提高动态web站点性能；
- Memcached数据结构简单（k-v），数据存放在内存里，并且是多线程服务、基于c/s架构，协议简单；
- Memcached基于libevent的事件处理、自主内存存储处理（slab allocation）；
- 其数据过期的方式有Lazy Expiration和LRU。

## Memcached特点

1. Memcached的数据流向如下图：

![memcached-flow](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/memcached_flow.png)

2. Slab allocation原理：

   - 将分配的内存分割成各种尺寸的块（chunk），并把尺寸相同的块分成组；
   - Memcached的内存分配以Page为单位，Page默认值为1M，可以再启动时通过`-I`参数来指定；
   - Slab是由多个Page组成的，Page按照指定的大小切割成多个chunk；
   - chunk的大小是由Growth Factor（增长因子）决定的。
   - Slab allocation的结构如下图：

   ![Slab allocation](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/slab_allocation.jpg)

3. Growth Factor：

   - Memcached在启动时通过`-f`选项可以指定Growth Factor因子，该值控制chunk大小的差异，默认值为1.25；
   - 通过memcached-tool命令可以查看指定Memcached实例的不同slab状态，可以看到各Item所占大小（chunk大小）差距为1.25；
   - 命令为`memcached-tool 127.0.0.1:11211 display`。

## Memcached数据过期方式

1. Lazy Expireation
   - Memcached内部不会监视记录是否过期，而是在get时查看记录的时间戳，检查记录是否过期；
   - 例如在创建一个k-v时，对其指定过期时间1小时，在服务器再次get此记录时，会对k-v的时间戳进行检查是否过期；
   - 这种技术被称为lazy（惰性）expiration。因此Memcached不会在过期监视上耗费CPU时间。
2. LRU
   - Memcached会优先使用已超时的记录的空间，但即使如此，也会发生追加新纪录时空间空间不足的情况，此时就要使用名为Least Recently Used（LRU）机制来分配空间；
   - 顾名思义，这是删除“最近最少使用”的记录的机制；
   - 当内存空间不足时（无法从Slab class获取到新的空间），就从最近未被使用的记录中搜索，并将其空间分配给新纪录，从缓存的实用角度看，该模型非常理想。

# 安装Memcached

- 使用yum安装Memcached，需要安装`libevent memcached libmemcached`三个包，由于memcached基于libevent进行事件处理，所以在安装memcached时，会自动安装libevent，而libmemcached软件包即使不安装，并不会影响Memcached的运行：

  ```bash
  $ yum install  libevent memcached libmemcached 
  ```

- 安装完成后，使用命令`systemctl start memcached`启动Memcached，然后可以查看Memcached的进程和端口：

  ```bash
  $ ps aux |grep memcached
  memcach+   5700  0.0  0.2 325608  1200 ?        Ssl  00:40   0:00 /usr/bin/memcached -u memcached -p 11211 -m 64 -c 1024
  ```

  - 其中`-u`指定了运行的用户，`-p`指定了运行的端口，`-m`分配了内存大小，单位为M，`-c`则是最大并发数。

  ```bash
  $ netstat -tlnp|grep memcached
  tcp        0      0 0.0.0.0:11211           0.0.0.0:*               LISTEN      5700/memcached      
  tcp6       0      0 :::11211                :::*                    LISTEN      5700/memcached 
  ```

- 对于Memcached的启动参数，可以自定义，但memcached并没有配置文件，所以可以采用以下两种方式更改：

  1. 手动输入命令启动，使用`/usr/bin/memcached`命令启动并指定启动参数；
  2. 编辑`/etc/sysconfig/memcached`文件，编辑文件的参数，文件内容如下：

  ```bash
  PORT="11211"	# 监听端口
  USER="memcached"	# 启动用户
  MAXCONN="1024"	# 最大并发数
  CACHESIZE="64"	# 分配内存大小
  OPTIONS=""	# 其他参数，如-l监听的主机
  ```

- 更多的启动参数，可以使用`memcached -h`查看。

# 查看memcached状态

- 在memcached运行时，我们需要查看其连接数、命中率等；

- Memcached自带了`memcached-tool`命令用来查看其运行状态，命令格式为`memcached-tool [ip:port] stats`：

  ```bash
  $ memcached-tool 127.0.0.1:11211 stats
  #127.0.0.1:11211   Field       Value
           accepting_conns           1
                 auth_cmds           0
               auth_errors           0
                     bytes           0
                bytes_read          59
             bytes_written         110
                cas_badval           0
                  cas_hits           0
                cas_misses           0
                 cmd_flush           0
                   cmd_get           0
                   cmd_set           0
                 cmd_touch           0
               conn_yields           0
     connection_structures          11
          curr_connections          10
                curr_items           0
                 decr_hits           0
               decr_misses           0
               delete_hits           0
             delete_misses           0
         evicted_unfetched           0
                 evictions           0
         expired_unfetched           0
                  get_hits           0
                get_misses           0
                hash_bytes      524288
         hash_is_expanding           0
          hash_power_level          16
                 incr_hits           0
               incr_misses           0
                  libevent 2.0.21-stable
            limit_maxbytes    67108864
       listen_disabled_num           0
                       pid        5700
              pointer_size          64
                 reclaimed           0
              reserved_fds          20
             rusage_system    0.035268
               rusage_user    0.003139
                   threads           4
                      time  1534265554
         total_connections          13
               total_items           0
                touch_hits           0
              touch_misses           0
                    uptime         744
                   version      1.4.15
  ```

  - 输出信息中，我们需要关注`get_hits`命中参数、`curr_items`目前存在memcached中的项目参数。

- 另外也可以使用命令`echo stats | nc 127.0.0.1 11211`，`nc`命令需要安装nmap-ncat软件包，或直接使用`yum install -y nc`，命令输出信息`memcached-tool`一致:

  ```bash
  $ echo stats |nc 127.0.0.1 11211
  STAT pid 5700
  STAT uptime 1152
  STAT time 1534265962
  STAT version 1.4.15
  STAT libevent 2.0.21-stable
  STAT pointer_size 64
  STAT rusage_user 0.004156
  STAT rusage_system 0.049873
  STAT curr_connections 10
  STAT total_connections 14
  STAT connection_structures 11
  STAT reserved_fds 20
  STAT cmd_get 0
  STAT cmd_set 0
  STAT cmd_flush 0
  STAT cmd_touch 0
  STAT get_hits 0
  STAT get_misses 0
  STAT delete_misses 0
  STAT delete_hits 0
  STAT incr_misses 0
  STAT incr_hits 0
  STAT decr_misses 0
  STAT decr_hits 0
  STAT cas_misses 0
  STAT cas_hits 0
  STAT cas_badval 0
  STAT touch_hits 0
  STAT touch_misses 0
  STAT auth_cmds 0
  STAT auth_errors 0
  STAT bytes_read 65
  STAT bytes_written 1137
  STAT limit_maxbytes 67108864
  STAT accepting_conns 1
  STAT listen_disabled_num 0
  STAT threads 4
  STAT conn_yields 0
  STAT hash_power_level 16
  STAT hash_bytes 524288
  STAT hash_is_expanding 0
  STAT bytes 0
  STAT curr_items 0
  STAT total_items 0
  STAT expired_unfetched 0
  STAT evicted_unfetched 0
  STAT evictions 0
  STAT reclaimed 0
  END
  ```

- 如果安装了`libmemcached`软件包，那么还可以使用`memstat`命令来查看memcached的运行状态，命令为`memstat --servers=[ip:port]`：

  ```bash
  [root@localhost ~]# memstat --servers=127.0.0.1:11211
  Server: 127.0.0.1 (11211)
  	 pid: 5700
  	 uptime: 1378
  	 time: 1534266188
  	 version: 1.4.15
  	 libevent: 2.0.21-stable
  	 pointer_size: 64
  	 rusage_user: 0.004191
  	 ...
  ```

  - 命令输出与memcached-tool基本一致。

---