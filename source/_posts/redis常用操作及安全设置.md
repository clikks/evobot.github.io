---
title: redis常用操作及安全设置
author: Evobot
date: 2018-08-20 20:27:41
categories: NoSQL
tags:
  - NoSQL
  - redis
image:
---

1. redis常用操作  
2. redis操作键值 
3. redis安全设置 

<!--more-->

---

# redis数据类型常用操作

## string操作

- 使用`set key1 value`创建一个string类型的记录时，如果再次对key1进行赋值，就会覆盖之前key1的值：

  ```bash
  127.0.0.1:6379> set key1 vaule1
  OK
  127.0.0.1:6379> GET key1
  "vaule1"
  127.0.0.1:6379> SET key1 value2
  OK
  127.0.0.1:6379> GET key1
  "value2"
  
  ```

- `SETNX [key] [value]`，这个命令的作用同样是创建一个键值，但如果创建的key存在，则返回0，如果不存在则返回1，并且创建key：

  ```bash
  127.0.0.1:6379> SETNX key1 value3
  (integer) 0
  127.0.0.1:6379> SETNX key2 value1
  (integer) 0
  127.0.0.1:6379> SETNX key_1 value1
  (integer) 1
  
  ```

- `SET [key] [value] EX`[seconds]，这个命令中EX用来给key设置过期时间，单位为秒，超过过期事件后，键值将被销毁：

  ```bash
  127.0.0.1:6379> SET key_2 abc EX 10
  OK
  127.0.0.1:6379> get key_2
  "abc"
  # 10s之后查看key_2的值已经不存在
  127.0.0.1:6379> get key_2
  (nil)
  
  ```

- `SETEX  [key] [seconds] [value]`，该条命令也是用来创建key并设置过期时间，当key已经存在时，会覆盖原有的值：

  ```bash
  127.0.0.1:6379> SETEX key_1 60 abcd
  OK
  127.0.0.1:6379> get key_1
  "abcd"
  # 60s后查看key_1,已不存在
  127.0.0.1:6379> get key_1
  (nil)
  
  ```

## list操作

- `LPUSH`是指从左侧开始，给list插入一个元素，即在redis中是从上面插入元素，新插入的元素序号为1：

  ```bash
  127.0.0.1:6379> LPUSH list1 abc
  (integer) 1
  127.0.0.1:6379> lpush list1 bbb
  (integer) 2
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "bbb"
  2) "abc"
  
  ```

- `LPOP`则是从左侧开始取出list中的元素，即从序号为1的元素开始取：

  ```bash
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "bbb"
  2) "abc"
  127.0.0.1:6379> LPOP list1
  "bbb"
  
  ```

- 相应的也有`RPOP`,`RPUSH`其作用则是从右侧开始取出或插入元素，即倒序取出或插入元素：

  ```bash
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "abc"
  127.0.0.1:6379> RPUSH list1 bbb
  (integer) 2
  127.0.0.1:6379> RPUSH list1 ccc
  (integer) 3
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "abc"
  2) "bbb"
  3) "ccc"
  
  127.0.0.1:6379> LPOP list1
  "abc"
  127.0.0.1:6379> RPOP list1
  "ccc"
  
  ```

- `LINSERT [key] BEFORE|AFTER [value1] [value2]`，这条命令是用来在list中指定位置插入元素，before表示指定的value1之前插入，after是指在value1之后插入元素，其中value1是在list中已存在的元素：

  ```bash
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "ccc"
  2) "aaa"
  3) "bbb"
  
  127.0.0.1:6379> LINSERT list1 BEFORE aaa 111
  (integer) 4
  
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "ccc"
  2) "111"
  3) "aaa"
  4) "bbb"
  
  127.0.0.1:6379> LINSERT list1 AFTER aaa 222
  (integer) 5
  
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "ccc"
  2) "111"
  3) "aaa"
  4) "222"
  5) "bbb"
  
  ```

- `LSET [key] [index] [value]`，该命令用来修改list中指定位置的元素的值，需要注意的是，这里的index索引号并不是LRANGE列出的序号，实际序号是从0开始：

  ```bash
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "ccc"
  2) "111"
  3) "aaa"
  4) "222"
  5) "bbb"
  127.0.0.1:6379> LSET list1 1 cccc
  OK
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "ccc"
  2) "cccc"
  3) "aaa"
  4) "222"
  5) "bbb"
  
  ```

- `LINDEX [key] [index]`，该命令用来查看指定索引位置的元素的值：

  ```bash
  127.0.0.1:6379> LINDEX list1 1
  "cccc"
  127.0.0.1:6379> LINDEX list1 0
  "ccc"
  
  ```

- `LLEN [key]`用来查看列表中元素个数：

  ```bash
  127.0.0.1:6379> LLEN list1
  (integer) 5
  127.0.0.1:6379> LRANGE list1 0 -1
  1) "ccc"
  2) "cccc"
  3) "aaa"
  4) "222"
  5) "bbb"
  
  ```

## set操作

- `SADD [seta] [value]`，该命令用于向集合seta中插入元素：

  ```bash
  127.0.0.1:6379> SADD seta aaa
  (integer) 1
  127.0.0.1:6379> SADD seta bbb
  (integer) 1
  
  ```

- `SMENBER [seta]`，用于查看集合中所有的元素：

  ```bash
  127.0.0.1:6379> SMEMBERS seta
  1) "aaa"
  2) "bbb"
  
  ```

- `SREM [seta] [value]`，用于删除集合中指定元素：

  ```bash
  127.0.0.1:6379> SREM seta aaa
  (integer) 1
  127.0.0.1:6379> SMEMBERS seta
  1) "bbb"
  
  ```

- `SPOP [seta]`，该命令会随机的从集合中取出一个元素，取出的元素将从集合中删除：

  ```bash
  127.0.0.1:6379> SPOP seta
  "bbb"
  127.0.0.1:6379> SMEMBERS seta
  (empty list or set)
  
  ```

- `SDIFF [seta] [setb]`，该命令用来计算seta和setb的差集，需要注意的是，计算出的差集是以命令中在前的集合为基准计算，即以前面的集合元素去跟后面的做比对，如下例：

  ```bash
  127.0.0.1:6379> SMEMBERS seta
  1) "aaa"
  2) "111"
  3) "ccc"
  4) "bbb"
  127.0.0.1:6379> SMEMBERS setb
  1) "111"
  2) "222"
  
  127.0.0.1:6379> SDIFF seta setb
  1) "aaa"
  2) "bbb"
  3) "ccc"
  127.0.0.1:6379> SDIFF setb seta
  1) "222"
  
  ```

- `SDIFFSTORE [setc] [seta] [setb]`，该命令是在求seta和setb的差集后将结果保存到setc中：

  ```bash
  127.0.0.1:6379> SDIFFSTORE setc seta setb
  (integer) 3
  127.0.0.1:6379> SMEMBERS setc
  1) "aaa"
  2) "bbb"
  3) "ccc"
  
  ```

- `SINTER [seta] [setb]`，该命令用来求交集，`SINTERSTORE setd seta setb则是求交集后将结果保存到setd中`：

  ```bash
  127.0.0.1:6379> SINTER seta setb
  1) "111"
  127.0.0.1:6379> SINTERSTORE setd seta setb
  (integer) 1
  127.0.0.1:6379> SMEMBERS setd
  1) "111"
  
  ```

- `SUNION [seta] [setb]`用来求并集，`SUNIONSTORE [sete] [seta] [setb]`则是保存并集结果到sete：

  ```bash
  127.0.0.1:6379> SUNION seta setb
  1) "bbb"
  2) "ccc"
  3) "111"
  4) "aaa"
  5) "222"
  127.0.0.1:6379> SUNIONSTORE sete seta setb
  (integer) 5
  127.0.0.1:6379> SMEMBERS sete
  1) "bbb"
  2) "ccc"
  3) "111"
  4) "aaa"
  5) "222"
  
  ```

- `SISMEMBER [seta] [value]`，该命令用来判断value是否属于集合seta，属于则返回1，否则返回0：

  ```bash
  127.0.0.1:6379> SISMEMBER seta aaa
  (integer) 1
  127.0.0.1:6379> SISMEMBER seta 222
  (integer) 0
  
  ```

- `SRANDMEMBER [seta] [num]`，该命令用来从集合中随机取值，但不会从集合中删除取出的元素，num是可选参数，指定取值数量：

  ```bash
  127.0.0.1:6379> SRANDMEMBER seta
  "111"
  127.0.0.1:6379> SRANDMEMBER seta
  "bbb"
  127.0.0.1:6379> SRANDMEMBER seta
  "ccc"
  
  127.0.0.1:6379> SRANDMEMBER seta 2
  1) "bbb"
  2) "ccc"
  127.0.0.1:6379> SRANDMEMBER seta 3
  1) "bbb"
  2) "ccc"
  3) "111"
  127.0.0.1:6379> SRANDMEMBER seta 2
  1) "ccc"
  2) "111"
  ```

## sort set操作

- `ZADD [zset] [score] [value]`，该命令用来创建有序集合，zset为集合的名称，score为权重：

  ```bash
  127.0.0.1:6379> ZADD zseta 2 aaa
  (integer) 1
  127.0.0.1:6379> ZADD zseta 5 bbb
  (integer) 1
  127.0.0.1:6379> ZRANGE zseta 0 -1
  1) "aaa"
  2) "bbb"
  127.0.0.1:6379> ZADD zseta 1 ccc
  (integer) 1
  127.0.0.1:6379> ZRANGE zseta 0 -1
  1) "ccc"
  2) "aaa"
  3) "bbb"
  
  ```

- `ZRANGE [zset] [start] [stop] [WITHSCORES]`，该命令是显示有序集合zset中指定序号的元素，可选参数`WITHSCORES表示显示权重score`：

  ```bash
  127.0.0.1:6379> ZRANGE zseta 0 -1 WITHSCORES
  1) "ccc"
  2) "1"
  3) "aaa"
  4) "2"
  5) "bbb"
  6) "5"
  
  ```

- `ZREM [zset] [value]`，该命令用来删除zset中指定的元素：

  ```bash
  127.0.0.1:6379> ZREM zseta ccc
  (integer) 1
  127.0.0.1:6379> ZRANGE zseta 0 -1
  1) "aaa"
  2) "bbb"
  
  ```

- `ZRANK [zset] [value] `，该命令用来查看元素的索引值，索引值从0开始，按照权重排序：

  ```bash
  127.0.0.1:6379> ZRANGE zseta 0 -1
  1) "aaa"
  2) "bbb"
  3) "sfgsdf"
  4) "asf"
  127.0.0.1:6379> ZRANK zseta bbb
  (integer) 1
  127.0.0.1:6379> ZRANK zseta asf
  (integer) 3
  
  ```

- `ZREVRANK [zset] [value]`，作用与上面的命令相同，但是是按照score倒序排列之后的索引值：

  ```bash
  127.0.0.1:6379> ZREVRANK zseta asf
  (integer) 0
  
  127.0.0.1:6379> ZREVRANK zseta bbb
  (integer) 2
  
  ```

- `ZREVRANGE [zset] [start] [stop] [WITHSCORES]` ，倒序排序输出指定范围的元素：

  ```bash
  127.0.0.1:6379> ZREVRANGE zseta 0 -1
  1) "asf"
  2) "sfgsdf"
  3) "bbb"
  4) "aaa"
  
  127.0.0.1:6379> ZREVRANGE zseta 0 -1 WITHSCORES
  1) "asf"
  2) "34"
  3) "sfgsdf"
  4) "23"
  5) "bbb"
  6) "5"
  7) "aaa"
  8) "2"
  
  ```

- `ZCARD [zset]`，该命令返回集合中元素的个数：

  ```bassh
  127.0.0.1:6379> ZCARD zseta
  (integer) 4
  
  ```

- `ZCOUNT [zset] [min] [max]`，该命令是输出权重score在指定范围min~max之间（包括min和max）的元素个数：

  ```bash
  127.0.0.1:6379> ZCOUNT zseta 1 20
  (integer) 2
  127.0.0.1:6379> ZCOUNT zseta 1 30
  (integer) 3
  
  ```

- `ZRANGEBYSCORE [zset] [min] [max] [WITHSCORES]`，该命令用来输出权重score在指定范围的元素：

  ```bash
  127.0.0.1:6379> ZRANGEBYSCORE zseta 1 20
  1) "aaa"
  2) "bbb"
  127.0.0.1:6379> ZRANGEBYSCORE zseta 1 30
  1) "aaa"
  2) "bbb"
  3) "sfgsdf"
  
  ```

- `ZREMRANGEBYRANK [zset] [start] [stop]`，该命令是用来删除指定索引范围的元素，索引按照score正向排序：

  ```bash
  127.0.0.1:6379> ZRANGE zseta 0 -1
  1) "aaa"
  2) "bbb"
  3) "sfgsdf"
  4) "asf"
  127.0.0.1:6379> ZREMRANGEBYRANK zseta 0 1
  (integer) 2
  127.0.0.1:6379> ZRANGE zseta 0 -1
  1) "sfgsdf"
  2) "asf"
  
  ```

- `ZREMRANGEBYSCORE [zset] [min] [max]`，该命令是删除指定权重范围的元素：

  ```bash
  127.0.0.1:6379> ZRANGE zseta 0 -1 withscores
  1) "sfgsdf"
  2) "23"
  3) "asf"
  4) "34"
  127.0.0.1:6379> ZREMRANGEBYSCORE zseta 20 30
  (integer) 1
  127.0.0.1:6379> ZRANGE zseta 0 -1 withscores
  1) "asf"
  2) "34"
  
  ```

## hash操作

- `HMSET [key] [field1] [value1] [field2] [value2]...` ，该命令用来批量建立哈希字段和对应的值：

  ```bash
  127.0.0.1:6379> HMSET hash_1 f1 aaa f2 bbb f3 ccc
  OK
  
  127.0.0.1:6379> HGETALL hash_1
  1) "f1"
  2) "aaa"
  3) "f2"
  4) "bbb"
  5) "f3"
  
  ```

- `HMGET [key]` ,该命令则是用来批量获取指定hash的多个字段的值：

  ```bash
  127.0.0.1:6379> HMGET hash_1 f1 f2 f3
  1) "aaa"
  2) "bbb"
  3) "ccc"
  
  ```

- `HDEL [key] [field]`：删除hash中指定字段：

  ```bash
  127.0.0.1:6379> HDEL hash_1 f1
  (integer) 1
  127.0.0.1:6379> HGETALL hash_1
  1) "f2"
  2) "bbb"
  3) "f3"
  4) "ccc"
  
  127.0.0.1:6379> HMGET hash_1 f1 f2 f3
  1) (nil)
  2) "bbb"
  3) "ccc"
  
  ```

- `HKEYS [key]`：打印指定hash所有的字段：

  ```bash
  127.0.0.1:6379> HKEYS hash_1
  1) "f2"
  2) "f3"
  
  ```

- `HVALS [key]`：打印指定hash所有的值：

  ```bash
  127.0.0.1:6379> HVALS hash_1
  1) "bbb"
  2) "ccc"
  
  ```

- `HLEN [key]`：打印指定hash的字段数量：

  ```bash
  127.0.0.1:6379> HLEN hash_1
  (integer) 2
  
  ```

# redis键值及服务常用操作

## 键值常用操作

- `keys *`：取出redis中所有的key，并且支持模糊匹配，用法为`keys set*`：

  ```bash
  127.0.0.1:6379> keys *
  1) "seta"
  2) "setc"
  3) "zseta"
  4) "hash_1"
  5) "sete"
  6) "setb"
  7) "setd"
  
  127.0.0.1:6379> keys set*
  1) "seta"
  2) "setc"
  3) "sete"
  4) "setb"
  5) "setd"
  ```

- `exists [key1] [key2] ...`：判断指定的key是否存在，存在则返回1，否则返回0：

  ```bash
  127.0.0.1:6379> EXISTS seta
  (integer) 1
  127.0.0.1:6379> EXISTS setf
  (integer) 0
  
  ```

- `del [key1] [key2] ...`：删除指定的key：

  ```bash
  127.0.0.1:6379> DEL seta
  (integer) 1
  127.0.0.1:6379> del sete setd
  (integer) 2
  
  ```

- `expire [key] [seconds]`：给指定的key设置过期时间，单位为秒：

  ```bash
  127.0.0.1:6379> EXPIRE mykey 5
  (integer) 1
  # 5s之后查看
  127.0.0.1:6379> get mykey
  (nil)
  
  ```

- `ttl [key]`：查看key的剩余过期时间，单位为秒，当key不存在时，返回-2，当key存在但没有设置过期时间时，返回-1：

  ```bash
  127.0.0.1:6379> EXPIRE zseta 20
  (integer) 1
  127.0.0.1:6379> ttl zseta
  (integer) 17
  127.0.0.1:6379> ttl zseta
  (integer) 15
  
  127.0.0.1:6379> ttl zseta
  (integer) -2
  127.0.0.1:6379> get zseta
  (nil)
  ```

- `select [index]`：先选择数据库，默认为0数据库，公有16个数据库：

  ```bash
  127.0.0.1:6379> select 1
  OK
  127.0.0.1:6379[1]> keys *
  (empty list or set)
  127.0.0.1:6379[1]> select 0
  OK
  127.0.0.1:6379> keys *
  1) "hash_1"
  2) "setb"
  3) "setc"
  
  ```

- `move [key] [db]`：将指定的key移动到指定的数据库中：

  ```bash
  127.0.0.1:6379> MOVE hash_1 1
  (integer) 1
  127.0.0.1:6379> select 1
  OK
  127.0.0.1:6379[1]> keys *
  1) "hash_1"
  
  ```

- `persist  [key]`：取消key的过期时间：

  ```bash
  127.0.0.1:6379> EXPIRE setb 60
  (integer) 1
  127.0.0.1:6379> ttl setb
  (integer) 57
  127.0.0.1:6379> PERSIST setb
  (integer) 1
  127.0.0.1:6379> ttl setb
  (integer) -1
  
  ```

- `randomkey`：随机返回一个key：

  ```bash
  127.0.0.1:6379> RANDOMKEY
  "setc"
  127.0.0.1:6379> RANDOMKEY
  "setb"
  
  ```

- `rename [key] [newkey]`：重命名指定key：

  ```bash
  127.0.0.1:6379> RENAME setb setbb
  OK
  127.0.0.1:6379> keys *
  1) "setbb"
  2) "setc"
  127.0.0.1:6379> get setb
  (nil)
  
  ```

- `type [key]`：查看key的数据类型：

  ```bash
  127.0.0.1:6379> TYPE setbb
  set
  127.0.0.1:6379> type hash_1
  hash
  127.0.0.1:6379> type list1
  list
  127.0.0.1:6379> type mykey
  string
  127.0.0.1:6379> type zkey
  zset
  ```

## 服务常用操作

- `dbsize`：返回当前数据库中key的数量：

  ```bash
  127.0.0.1:6379> dbsize
  (integer) 6
  127.0.0.1:6379> keys *
  1) "hash_1"
  2) "zkey"
  3) "setbb"
  4) "setc"
  5) "list1"
  6) "mykey"
  
  ```

- `info`：返回redis数据库状态信息：

  ```bash
  127.0.0.1:6379> info
  # Server
  redis_version:4.0.11
  redis_git_sha1:00000000
  redis_git_dirty:0
  redis_build_id:27b5b45d01df6502
  redis_mode:standalone
  os:Linux 3.10.0-693.21.1.el7.x86_64 x86_64
  arch_bits:64
  multiplexing_api:epoll
  atomicvar_api:atomic-builtin
  gcc_version:4.8.5
  process_id:32693
  run_id:2235d6644632c1a4802b2eddf4cd646ba352be69
  tcp_port:6379
  uptime_in_seconds:174662
  uptime_in_days:2
  hz:10
  lru_clock:8142204
  executable:/usr/local/src/redis-4.0.11/redis-server
  config_file:/etc/redis.conf
  ...
  ```

- `flushall`：清空数据库中所有的key，包括其他所有数据库；

- `flushdb`：清空当前数据库所有的key：

  ```bash
  127.0.0.1:6379[1]> keys *
  1) "key_1"
  127.0.0.1:6379[1]> FLUSHDB
  OK
  127.0.0.1:6379[1]> keys *
  (empty list or set)
  
  ```

- `bgsave`：后台保存数据到RDB数据文件中；`save`：前台保存数据到RDB数据文件中：

  ```bash
  127.0.0.1:6379> BGSAVE
  Background saving started
  127.0.0.1:6379> save
  OK
  
  ```

- `config get *`：获取所有配置参数；`config get [para]`：获取指定的配置参数：

  ```bash
  127.0.0.1:6379> CONFIG GET dir
  1) "dir"
  2) "/data/redis"
  127.0.0.1:6379> CONFIG GET port
  1) "port"
  2) "6379"
  127.0.0.1:6379> CONFIG GET *
    1) "dbfilename"
    2) "dump.rdb"
    3) "requirepass"
    4) ""
    5) "masterauth"
    6) ""
    7) "cluster-announce-ip"
    8) ""
    9) "unixsocket"
   10) ""
   11) "logfile"
   12) "/var/log/redis.log"
   13) "pidfile"
   14) "/var/run/redis_6379.pid"
  ...
  ```

- `config set [para]`：更改指定配置参数：

  ```bash
  127.0.0.1:6379> CONFIG GET timeout
  1) "timeout"
  2) "0"
  127.0.0.1:6379> CONFIG SET timeout 60
  OK
  127.0.0.1:6379> CONFIG GET timeout
  1) "timeout"
  2) "60"
  
  ```

- 数据恢复：首先定义或者确定dir目录以及dbfilename，然后把备份的数据文件放到dir目录下，重启redis服务即可自动恢复数据：

  ```bash
  127.0.0.1:6379> CONFIG GET dir
  1) "dir"
  2) "/data/redis"
  127.0.0.1:6379> CONFIG GET dbfilename
  1) "dbfilename"
  2) "dump.rdb"
  
  ```

# redis安全设置

1. 设置监听ip：在配置文件中的`bind`配置项中，绑定监听的ip为内网ip或指定的公网ip，多个ip使用空格分割；防止redis被登陆后通过`config set dir`的形式将黑客的公钥写入到/root/.ssh/authorized_keys文件中去：

   ```bash
   bind 127.0.0.1 10.139.151.2
   ```

2. 设置监听端口：修改`port`配置项，将其改为不常用的端口号：

   ```bash
   port 4192
   ```

   - 配置后重启redis服务，下次登陆时需要使用`redis-cli -p [port]`连接指定端口登陆redis，否则默认还是会使用6379连接redis。

3. 设置密码：在配置文件中找到`requirepass`配置项，在后面加上密码：

   ```bash
   requirepass this>is>passwd
   ```

   - 保存配置后重启redis，使用`redis-cli -a [passwd]`登陆redis，否则也能够登陆到redis，但无法执行命令：

   ```bash
   $ redis-cli -p 4192
   127.0.0.1:4192> keys *
   (error) NOAUTH Authentication required.
   
   ```

   ```bash
   [root@evobot ~]# redis-cli -p 4192 -a 'this>is>passwd'
   Warning: Using a password with '-a' option on the command line interface maynot be safe.
   127.0.0.1:4192> keys *
   1) "zkey"
   2) "setbb"
   3) "mykey"
   4) "setc"
   5) "hash_1"
   
   ```

4. 将redis的`CONFIG`命令重命名：在配置文件中查找`rename-command`配置项，写入`rename-command CONFIG EVOBOT`，将CONFIG命令改名，然后重启redis服务：

   ```bash
   127.0.0.1:4192> CONFIG GET dir
   (error) ERR unknown command `CONFIG`, with args beginning with: `GET`, `dir`,
   127.0.0.1:4192> evobot get dir
   1) "dir"
   2) "/data/redis"
   
   ```

5. 禁用`CONFIG`命令，则直接将配置文件改为`rename-command CONFIG ""`即可。

---

