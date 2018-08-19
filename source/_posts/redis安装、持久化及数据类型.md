---
title: redis安装、持久化及数据类型
author: Evobot
date: 2018-08-19 22:32:00
categories: NoSQL
tags:
  - NoSQL
  - redis
image:
---

1. redis介绍  
2. redis安装 
3. redis持久化 
4. redis数据类型 

<!--more-->

---

# Redis介绍

- Redis和Memcached类似，也属于k-v数据存储，但redis支持更多value类型，除了k-v的string类型外，还支持hash、lists（链表）、sets（集合）和sorted sets（有序集合）；
- redis使用了两种文件格式：全量数据（RDB）和增量请求（aof）；
  - 全量数据格式是把内存中的数据写入磁盘，便于下次读取文件进行加载；
  - 增量请求文件 则是把内存中的数据序列化为操作请求，用于读取文件进行replay得到数据，这种类似于mysql binlog；
- redis的存储分为内存存储、磁盘存储(RDB)和log文件(aof)三个部分。

# redis安装

## 下载安装

- 首先下载稳定版的redis安装包，到[redis官网](https://redis.io/)下载，这里使用的是4.0.11版本；

- 然后解压安装包，进行编译安装，redis编译不需要`./configure`步骤，直接`make && make install`即可；

- 安装完成后，复制配置文件：

  ```bash
  $ cp redis.conf /etc/ 
  ```

## redis配置文件

- 打开配置文件，找到`daemonize no`配置项，将其改为**yes**，这个选项表示将redis启动模式改为后台启动，否则redis默认以前台启动方式启动；

- 然后找到`logfile`配置项，指定redis的日志保存路径：

  ```bash
  logfile "/var/log/redis.log"
  ```

- redis存在库的概念，默认有16个库，从0-15，默认在0库里；在选项`databases 16`可以看到；

- `rdbcompression yes`配置表示是否压缩RDB文件，默认yes表示压缩；

- `dbfilename dump.rdb`配置指定rdb的文件名；

- `dir ./`配置则是指定RDB文件存储位置，默认为安装目录，建议更改为其他路径，另外aof文件也会放在这个目录，如：

  ```bash
  dir /data/redis/
  ```

- `appendonly no`配置，是用来指定是否开启aof配置的，默认为no不开启，`appendfilename "appendonly.aof"`配置则是指定aof日志的文件名，生产的文件会保存在之前的`dir`配置项配置的目录里；

- `appendfsync everysec`配置是指定什么时候记录aof日志，`everysec`表示每秒记录，`always`则表示总是记录，即每执行一个操作就记录一次，而no则是根据系统算法进行记录，不够安全，一般使用everysec配置；

## 启动redis

- 启动redis前，可以修改以下两个内核参数，不修改内核参数，redis同样能够正常工作，但会在日志中产生告警，所以建议修改内核参数：

  ```bash
  sysctl vm.overcommit_memory=1
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
  ```

- 另外定义高负载的环境，还需要修改`/proc/sys/net/core/somaxconn`每个端口最大的监听队列的长度 参数，系统默认为128，建议修改为1024或2048：

  ```bash
  echo 2048 > /proc/sys/net/core/somaxconn
  ```

- 命令行中执行上面的命令只是临时生效，永久生效需要将上面的命令写入`/etc/rc.local`文件中；

- 然后启动redis服务，使用命令`redis-server /etc/redis.conf`：

  ```bash
  $ redis-server /etc/redis.conf
  
  $ ps aux |grep redis
  root     32082  0.0  0.1 145264  2172 ?        Ssl  23:49   0:00 redis-server 127.0.0.1:6379
  root     32106  0.0  0.0 112676   984 pts/0    R+   23:49   0:00 grep --color=auto redis
  
  ```

# redis持久化

## redis持久化介绍

- redis提供了两种持久化的方式，分别为RDB（Redis DataBase）和AOF(Append Only File)；
- 如果关闭RDB和AOF，那么redis的数据则都存储在内存中，一旦服务器关闭或redis服务关闭，则redis中存储的数据都会丢失；
- RDB，就是在不同的时间点，将redis存储的数据生成快照病存储到磁盘等介质上；
- AOF，则是将redis执行过的所有写指令记录下来，在下次redis重新启动时，只需要将这些写指令从前到后再重复执行一遍，就可以实现数据恢复；
- RDB和AOF两种方式可以同时使用，在这种情况下，redis重启时会优先采用AOF方式进行数据恢复，这是因为AOF方式恢复的数据完整度更高；
- 如果没有数据持久化需求，则可以关闭RDB和AOF，这样redis就和memcached相同，成为纯内存数据库

## redis持久化配置

- redis的RDB方式，存储到磁盘上的时间，是由配置文件中的`save`配置项指定的，如果关闭RDB，则注释`save`的三行配置，打开`save ""`配置即可，下面是默认redis的RDB写入磁盘的配置：

  ```bash
  save 900 1	# 表示900秒更改了一次
  save 300 10	# 表示300秒更改了10次
  save 60 10000	# 60秒发生了10000次更改
  ```

  - 以上3个配置，只要满足一项，redis就会将数据写入磁盘，这里的更改是指增加、删除、修改操作。
  - 一般保持默认配置即可满足大部分需求。

- AOF会随着redis的运行时间，存储的文件也会也来越大，因为AOF并不会删除已经过期的记录，AOF的配置在配置文件中就是`appendfsync`配置项。

# redis数据类型

## string类型

- string为最简单的类型，与memcached一样的类型，一个key对应一个value，支持的操作与memcached操作类似，但功能更丰富，设置可以存二进制的对象；

- 使用redis-cli命令进行redis的命令行工具，使用`set [key][value]`增加一个键值记录：

  ```bash
  [root@evobot ~]# redis-cli
  127.0.0.1:6379> SET mykey 123
  OK
  127.0.0.1:6379> SET mykey2 "evobot"
  OK
  127.0.0.1:6379>
  
  ```

- 使用`get [key]`获取一个键的值：

  ```bash
  127.0.0.1:6379> GET mykey
  "123"
  127.0.0.1:6379> get mykey2
  "evobot"
  ```

- 使用`mset [key1][value1] [key2][value2] ...`可以一次增加多个k-v记录，使用`mget [key1] [key2] ...`可以一次获取多个键的值：

  ```bash
  127.0.0.1:6379> mset key1 "evobot" key2 "hell world" key3 "myredis"
  OK
  
  127.0.0.1:6379> mget key1 key2 key3
  1) "evobot"
  2) "hell world"
  3) "myredis"
  
  ```

## list类型

- list是一个链表结构，主要功能是push、pop、获取一个范围所有值等，操作中的key理解为链表的名字；

- 使用list结构，可以轻松实现最新消息排列等功能，如微博的TimeLine，list的另一个作用就是消息队列，可以利用list的push操作，将任务存在list中，然后工作线程再用pop操作将任务取出后进行执行；

- 对于list类型的数据操作如下：

  - PUSH操作

  ```bash
  127.0.0.1:6379> LPUSH list1 "evobot"
  (integer) 1
  127.0.0.1:6379> LPUSH list1 "second value"
  (integer) 2
  127.0.0.1:6379> LPUSH list1 "third value"
  (integer) 3
  
  ```

  - LRANGE操作，查看list中的数据：

  ```bash
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "third value"
  2) "second value"
  3) "evobot"
  127.0.0.1:6379> LRANGE list1 0 1
  1) "third value"
  2) "second value"
  
  ```

  - POP操作：

  ```bash
  127.0.0.1:6379> LPOP list1
  "third value"
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "second value"
  2) "evobot"
  # pop取出后，原list中就没有了最后一个值
  ```

  > 这里的LPUSH、LRANGE、LPOP中的L都是指left，即从左往右正向取值。

## set类型

- set是集合，与数学概念中的集合类似，对集合的操作有添加、删除元素，有多个集合求交并差等操作；

- 操作中key理解为集合的名字，例如在微博应用中，可以将一个用户所有的关注人存在一个集合中，将其所有的粉丝存在另一个集合中，因为redis提供了求交集、并集、差集等操作，就可以非常方便的实现如共同关注、共同喜好、二度好友等功能；

- 对于集合的操作，还可以使用不同的命令，将结果返回客户端或存到一个新的集合中；

- set类型的操作如下：

  - `SADD [setname] [value]`为集合增加元素：

  ```bash
  127.0.0.1:6379> SADD set1 a
  (integer) 1
  127.0.0.1:6379> SADD set1 b
  (integer) 1
  127.0.0.1:6379> SADD set1 c
  (integer) 1
  127.0.0.1:6379> SADD set1 d
  (integer) 1
  
  127.0.0.1:6379> SADD set2 a
  (integer) 1
  127.0.0.1:6379> SADD set2 1
  (integer) 1
  127.0.0.1:6379> SADD set2 e
  (integer) 1
  
  ```

  - `SMEMBERS [setname]`查看集合的所有元素：

  ```bash
  127.0.0.1:6379> SMEMBERS set1
  1) "b"
  2) "a"
  3) "d"
  4) "c"
  
  127.0.0.1:6379> SMEMBERS set2
  1) "1"
  2) "e"
  3) "a"
  
  ```

  - `SUNION [set1] [set2]`并集操作：

  ```bash
  127.0.0.1:6379> SUNION set1 set2
  1) "1"
  2) "d"
  3) "c"
  4) "e"
  5) "b"
  6) "a"
  
  ```

  - `SINTER [set1] [set2]`交集操作：

  ```bash
  127.0.0.1:6379> SINTER set1 set2
  1) "a"
  ```

  - `SDIFF [set1] [set2]`差集操作：

  ```bash
  127.0.0.1:6379> SDIFF set1 set2
  1) "b"
  2) "c"
  3) "d"
  ```

  - `SREM [setname] [value]`删除集合中元素：

  ```bash
  127.0.0.1:6379> SREM set1 c
  (integer) 1
  127.0.0.1:6379> SMEMBERS set1
  1) "b"
  2) "a"
  3) "d"
  
  ```

## sort set类型

- sort set是有序集合，它比set多了一个权重参数score，使得集合中的元素能够按照score进行有序排列；

- 例如存储一个班的同学成绩的sorted sets，其集合value可以是同学的学号，而score就可以是其考试得分，这样数据在插入集合时，就已经进行了天然的排序；

- sort set类型的操作如下：

  - `ZADD [setname] [score] [value]`为sort set添加有序集合和值：

  ```bash
  127.0.0.1:6379> ZADD set3 12 abc
  (integer) 1
  127.0.0.1:6379> ZADD set3 2 "abc123"
  (integer) 1
  127.0.0.1:6379> ZADD set3 24 "evobot"
  (integer) 1
  127.0.0.1:6379> ZADD set3 4 "string"
  (integer) 1
  ```

  - `ZRANGE [setname] [start] [stop]`查看从start开始到stop结束范围内的有序集合的值：

  ```bash
  127.0.0.1:6379> ZRANGE set3 0 -1
  1) "abc123"
  2) "string"
  3) "abc"
  4) "evobot"
  
  ```

  > ZRANGE查看有序集合的值的时候，score不会输出，而值的排列时按照score大小进行有序排列输出的。

  - `ZREVRANGE [setname] [start] [stop]`，反序输出集合的值：

  ```bash
  127.0.0.1:6379> ZREVRANGE set3 0 -1
  1) "evobot"
  2) "abc"
  3) "string"
  4) "abc123"
  
  ```

## hash类型

- 在实际使用中，我们经常将一些结构化的信息打包成hashmap，在客户端序列化后存储为一个字符串的值（一般为JSON格式），如用户的昵称、年龄、性别、积分等；

- 对于hash类型的操作如下：

  - `HSET [hashname] [field] [value]`，添加字段field的值value到名为hashname哈希字段中：

  ```bash
  127.0.0.1:6379> HSET hash1 name evobot
  (integer) 1
  127.0.0.1:6379> HSET hash1 age 25
  (integer) 1
  127.0.0.1:6379> HSET hash1 job it
  (integer) 1
  
  ```

  - `HGET [hashname] [field]`获取指定哈希的字段的值：

  ```bash
  127.0.0.1:6379> HGET hash1 name
  "evobot"
  127.0.0.1:6379> HGET hash1 age
  "25"
  127.0.0.1:6379> HGET hash1 job
  "it"
  
  ```

  - `HGETALL [hashname]`获取指定哈希的所有字段及其对应的值，输出中奇数行为字段名，偶数行为字段的值：

  ```bash
  127.0.0.1:6379> HGETALL hash1
  1) "name"
  2) "evobot"
  3) "age"
  4) "25"
  5) "job"
  6) "it"
  
  ```

---