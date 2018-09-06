---
title: MongoDB分片与备份恢复
author: Evobot
date: 2018-09-05 22:35:10
categories: NoSQL
tags:
  - MongoDB
image:
---

1. mongodb分片介绍
2.  mongodb分片搭建
3. mongodb分片测试
4. mongodb备份恢复

<!--more-->

---

# MongoDB分片

## 分片介绍

- 分片就是将数据库进行拆分，将大型集合分割到不同服务器上，比如，本来100G的数据，可以分割成10份存储到10台服务器上，这样每台机器只有10G数据；
- 通过一个mongos的进程（路由）实现分片后的数据存储和访问，也就是说mongos是整个分片架构的核心，对客户端而言是不知道是否有分片的，客户端只需要把读写操作转达给mongos即可，不再发送的mongod；
- 虽然分片会把数据分割到很多台服务器上，但是每个节点（副本集）都是需要有一个备用角色的，这样能够保证数据的高可用；
- 当系统需要更多空间或资源的时候，分片可以让我们按需方便扩展，只需要吧mongodb服务的机器加入到分片集群中即可；

- MongoDB分片架构图：

  ![i9Z3md.png](https://s1.ax1x.com/2018/09/05/i9Z3md.png)

  - 图中不论是Router还是Config Servers、Shard，都是由副本集构成的，以实现高可用。

- mongos：数据库集群请求的入口，所有的请求都通过mongos进行协调，不需要在应用程序中添加一个路由选择器，mongos自己就是一个请求分发中心，它负责把对应的数据请求转发到对应的Shard服务器上，在生产环境中，通常有多mongos作为请求的入口，防止其中一个发生宕机导致所有的mongodb请求都没办法操作；

- config server：配置服务器，存储所有数据库元信息（路由、分片）的配置，mongos本身没有物理存储分片服务器和数据路由信息，只是缓存在内存里，配置服务器则实际存储这些数据，mongos第一次启动或者重启，就会从config server中加载配置信息，以后如果配置服务器信息变化，会通知到所有的mongos更新自己的状态，这样mongos就能继续准确路由，在生产环境中通常有多个config server配置服务器，因为它存储了分片路由的元数据，防止数据丢失；

- shard：存储了一个集合部分数据的MongoDB实例，每个分片是单独的MongoDB服务或者副本集，在生产环境中，所有的分片都应该是副本集。

## 分片搭建

### 服务器规划

- 准备三台机器A、B、C
- A搭建：mongos、config server、副本集1主节点、副本集2仲裁、副本集3从节点；
- B搭建：mongos、config server、副本集1从节点、副本集2主节点、副本集3仲裁；
- C搭建：mongos、config server、副本集1仲裁、副本集2从节点、副本集3主节点
- 端口分配：mongos 20000，config 21000，副本集1 27001，副本集2 27002，副本集3 27003
- 三台机器全部关闭firewalld和selinux服务，或者增加对应端口规则。

### 目录规划

- 分别在三台机器上创建各个角色所需的目录：

  ```bash
  mkdir -p /data/mongodb/mongos/log
  mkdir -p /data/mongodb/config/{data,log}
  mkdir -p /data/mongodb/shard1/{data,log}
  mkdir -p /data/mongodb/shard2/{data,log}
  mkdir -p /data/mongodb/shard3/{data,log}
  ```

### config server配置

- 为三台机器添加config server配置文件：

  ```bash
  mkdir /etc/mongod/
  vim /etc/mongod/config.conf
  ```

  写入如下内容：

  ```bash
  pidfilepath = /var/run/mongodb/configsrv.pid
  dbpath = /data/mongodb/config/data/
  logpath = /data/mongodb/config/log/configsrv.log
  logappend = true
  bind_ip = 0.0.0.0 #可以绑定各机器自己的内网ip
  port = 21000
  fork = true
  configsvr = true	#declare this is a config db of cluster;
  replSet=configs #副本集名称
  maxConns=20000	#最大连接数
  ```

- 然后分别启动三台config server服务：

  ```bash
  $ mongod -f /etc/mongod/config.conf 
  about to fork child process, waiting until server is ready for connections.
  forked process: 6448
  child process started successfully, parent exiting
  
  $ ps aux |grep mongod
  root       6448 11.0  9.8 1069924 47524 ?       Sl   22:28   0:00 mongod -f /etc/mongod/config.conf
  
  $ netstat -tlnp|grep mongod
  tcp        0      0 192.168.49.128:21000    0.0.0.0:*               LISTEN      6448/mongod   
  ```

- 登录任意一台机器的21000端口，初始化config server副本集：

  ```bash
  $ mongo --port 21000 --host 192.168.49.128
  
  > config={_id:"configs",members:[{_id:0,host:"192.168.49.128:21000"},{_id:1,host:"192.168.49.129:21000"},{_id:2,host:"192.168.49.130:21000"}]}
  
  > rs.initiate(config)
  {
  	"ok" : 1,
  	"operationTime" : Timestamp(1536245789, 1),
  	"$gleStats" : {
  		"lastOpTime" : Timestamp(1536245789, 1),
  		"electionId" : ObjectId("000000000000000000000000")
  	},
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536245789, 1),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  
  ```

### 分片(shard)配置

- 在三台服务器上添加配置文件`vi /etc/mongod/shard1.conf`，写入以下内容：

  ```bash
  pidfilepath = /var/run/mongodb/shard1.pid
  dbpath = /data/mongodb/shard1/data/
  logpath = /data/mongodb/shard1/log/shard1.log
  logappend = true
  bind_ip = 0.0.0.0
  port = 27001
  fork = true
  httpinterface = true	#打开web监控，3.6版本后已删除此配置参数
  rest = true	# 3.6之后版本需要删除此配置项
  replSet = shard1	#副本集名称
  shardsvr = true
  maxConns = 20000
  
  ```

- 然后将三台服务器上的shard1的配置文件拷贝为shard2.conf和shard3.conf，并修改对应的配置：

  ```bash
  $ cp shard1.conf shard2.conf 
  $ cp shard1.conf shard3.conf 
  $ sed -i 's/shard1/shard2/g;s/27001/27002/g' shard2.conf
  $ sed -i 's/shard1/shard3/g;s/27001/27003/g' shard3.conf
  ```

- 接着在三台服务器上启动shard1：

  ```bash
  //启动shard1
  $ ps aux |grep mongod
  root       6448  0.5 11.9 1580640 57964 ?       Sl   22:28   0:23 mongod -f /etc/mongod/config.conf
  root       6646  4.7  9.8 1028944 47448 ?       Sl   23:40   0:00 mongod -f /etc/mongod/shard1.conf
  root       6672  0.0  0.2 112724   976 pts/1    R+   23:41   0:00 grep --color=auto mongod
  $ netstat -tlnp|grep mongod
  tcp        0      0 0.0.0.0:27001           0.0.0.0:*               LISTEN      6646/mongod         
  tcp        0      0 192.168.49.128:21000    0.0.0.0:*               LISTEN      6448/mongod    
  ```

- 登录128或129的mongo命令行，配置副本集，130由于作为shard1的仲裁，所以不能在130机器上进行配置：

  ```json
  //配置副本集
  $ mongo --port 27001
  > use admin
  > config={_id:"shard1",members:[{_id:0,host:"192.168.49.128:27001"},{_id:1,host:"192.168.49.129:27001"},{_id:2,host:"192.168.49.130：27001",arbiterOnly:true}]}
  > rs.initiate(config)
  ```

- shard2副本集配置：

  ```json
  $ mongod -f /etc/mongod/shard2.conf
  $ mongo --port 27002
  > use admin
  > config={_id:"shard2",members:[{_id:0,host:"192.168.49.128:27002",arbiterOnly:true},{_id:1,host:"192.168.49.129:27002"},{_id:3,host:"192.168.49.130:27002"}]}
  > rs.initiate(config)
  ```

- shard3副本集配置：

  ```bash
  $ mongod -f /etc/mongod/shard2.conf
  $ mongo --port 27002
  > use admin
  > config={_id:"shard2",members:[{_id:0,host:"192.168.49.128:27003"},{_id:1,host:"192.168.49.129:27003",arbiterOnly:true},{_id:3,host:"192.168.49.130:27003"}]}
  > rs.initiate(config)
  ```

### 路由服务器(mongos)配置

- 在三台服务器上添加配置文件`vi /etc/mongod/mongos.conf`,写入如下内容：

  ```bash
  pidfilepath = /var/run/mongodb/mongos.pid
  logpath = /data/mongodb/mongos/log/mongos.log
  logappend = true
  bind_ip = 0.0.0.0
  port = 20000
  fork = true
  configdb = configs/192.168.49.128:21000,192.168.49.129:21000,192.168.49.130:21000	#监听的配置服务器，只能有1个或者3个，configs为配置服务器的副本集名字
  maxConns = 20000
  
  ```

- 启动mongos服务，启动命令与config server和shard不同：

  ```bash
  mongos -f /etc/mongod/mongos.conf
  ```

  ```bash
  $ ps aux |grep mongos
  root       6927  1.5  2.6 255320 12676 ?        Sl   00:12   0:00 mongos -f /etc/mongod/mongos.conf
  root       6952  0.0  0.2 112724   976 pts/1    D+   00:12   0:00 grep --color=auto mongos
  $ netstat -tlnp |grep mongos
  tcp        0      0 0.0.0.0:20000           0.0.0.0:*               LISTEN      6927/mongos  
  ```

- 然后登录任何一台机器的20000端口，将所有的分片和路由串联：

  ```bash
  $ mongo --port 20000
  mongos> use admin
  mongos> sh.addShard("shard1/192.168.49.128:27001,192.168.49.129:27001,192.168.49.130:27001")
  mongos> sh.addShard("shard2/192.168.49.128:27002,192.168.49.129:27002,192.168.49.130:27002")
  mongos> sh.addShard("shard3/192.168.49.128:27003,192.168.49.129:27003,192.168.49.130:27003")
  
  ```

- 查看集群状态使用`rs.status()`命令：

  ```bash
  mongos> sh.status()
  --- Sharding Status --- 
    sharding version: {
    	"_id" : 1,
    	"minCompatibleVersion" : 5,
    	"currentVersion" : 6,
    	"clusterId" : ObjectId("5b91402a5d2d80280a48ee8c")
    }
    shards:
          {  "_id" : "shard1",  "host" : "shard1/192.168.49.128:27001,192.168.49.129:27001",  "state" : 1 }
          {  "_id" : "shard2",  "host" : "shard2/192.168.49.129:27002,192.168.49.130:27002",  "state" : 1 }
          {  "_id" : "shard3",  "host" : "shard3/192.168.49.128:27003,192.168.49.130:27003",  "state" : 1 }
    active mongoses:
          "3.6.7" : 1
    autosplit:
          Currently enabled: yes	#当前分片状态
    balancer:
          Currently enabled:  yes	#当前负载均衡状态
          Currently running:  no	# 由于没有存入数据，所以运行状态为no
          Failed balancer rounds in last 5 attempts:  0
          Migration Results for the last 24 hours: 
                  No recent migrations
    databases:
          {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                  config.system.sessions
                          shard key: { "_id" : 1 }
                          unique: false
                          balancing: true
                          chunks:
                                  shard1	1
                          { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : shard1 Timestamp(1, 0) 
  
  ```

## 分片测试

- 登录任何一台服务器的mongo 20000端口；

- 执行命令`db.runCommand({enablesharding:"testdb"})`或者`sh.enableSharding("testdb")`，用来指定要分片的数据库，如果数据库不存在，会自动创建：

  ```bash
  mongos> db.runCommand({enablesharding:"testdb"})
  {
  	"ok" : 1,
  	"operationTime" : Timestamp(1536251270, 8),
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536251270, 8),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  
  ```

- 执行`db.runCommand({shardcollection:"testdb.table1",key:{id:1}})`或者`sh.shardCollection("testdb.table1",{"id":1})`，用来指定数据库里需要分片的集合和片键：

  ```bash
  mongos> sh.shardCollection("testdb.table1",{"id":1})
  {
  	"collectionsharded" : "testdb.table1",
  	"collectionUUID" : UUID("ce310fe8-9c4e-4d0a-8b4a-b0ee39940352"),
  	"ok" : 1,
  	"operationTime" : Timestamp(1536251408, 14),
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536251408, 14),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  
  ```

- 然后为testdb数据库插入10000条数据：

  ```bash
  mongos> use testdb
  switched to db testdb
  mongos> for (var i=1;i<10000;i++) db.table1.save({id:i,"test1":"testval1"})
  WriteResult({ "nInserted" : 1 })
  
  mongos> db.table1.stats()	#查看table1状态
  
  ```

- 执行`sh.status()`可以看到testdb所在的分片：

  ```bash
  mongos> sh.status()
  --- Sharding Status --- 
    sharding version: {
    	"_id" : 1,
    	"minCompatibleVersion" : 5,
    	"currentVersion" : 6,
    	"clusterId" : ObjectId("5b91402a5d2d80280a48ee8c")
    }
    shards:
          {  "_id" : "shard1",  "host" : "shard1/192.168.49.128:27001,192.168.49.129:27001",  "state" : 1 }
          {  "_id" : "shard2",  "host" : "shard2/192.168.49.129:27002,192.168.49.130:27002",  "state" : 1 }
          {  "_id" : "shard3",  "host" : "shard3/192.168.49.128:27003,192.168.49.130:27003",  "state" : 1 }
    active mongoses:
          "3.6.7" : 1
    autosplit:
          Currently enabled: yes
    balancer:
          Currently enabled:  yes
          Currently running:  no
          Failed balancer rounds in last 5 attempts:  0
          Migration Results for the last 24 hours: 
                  No recent migrations
    databases:
          {  "_id" : "config",  "primary" : "config",  "partitioned" : true }
                  config.system.sessions
                          shard key: { "_id" : 1 }
                          unique: false
                          balancing: true
                          chunks:
                                  shard1	1
                          { "_id" : { "$minKey" : 1 } } -->> { "_id" : { "$maxKey" : 1 } } on : shard1 Timestamp(1, 0) 
          {  "_id" : "testdb",  "primary" : "shard2",  "partitioned" : true }
                  testdb.table1
                          shard key: { "id" : 1 }
                          unique: false
                          balancing: true
                          chunks:
                                  shard2	1	#所在shard2
                          { "id" : { "$minKey" : 1 } } -->> { "id" : { "$maxKey" : 1 } } on : shard2 Timestamp(1, 0) 
  
  ```

# MongoDB备份恢复

## 备份

- 备份指定库：

  ```bash
  mongodump --host [ip] --port [port] -d [db] -o [/path/to/dir/]
  ```

  ```bash
  $ mkdir /tmp/mongobak 
  $ mongodump --host 127.0.0.1 --port 20000 -d testdb -o /tmp/mongobak/
  2018-09-07T00:44:38.132+0800	writing testdb.table1 to 
  2018-09-07T00:44:38.247+0800	done dumping testdb.table1 (9999 documents)
  
  # 会生成对应数据库同名的目录，并生成两个备份文件
  $ ls /tmp/mongobak/testdb/
  table1.bson  table1.metadata.json
  
  ```

- 备份所有库：

  ```bash
  mongodump --host [ip] --port [port] -o [/path/to/dir/]
  ```

  ```bash
  $ mongodump --host 127.0.0.1 --port 20000 -o /tmp/mongobak/
  2018-09-07T00:48:30.744+0800	writing admin.system.version to 
  2018-09-07T00:48:30.747+0800	done dumping admin.system.version (1 document)
  2018-09-07T00:48:30.747+0800	writing testdb.table1 to 
  2018-09-07T00:48:30.747+0800	writing db2.tb1 to 
  2018-09-07T00:48:30.748+0800	writing db2.table1 to 
  2018-09-07T00:48:30.748+0800	writing config.locks to 
  2018-09-07T00:48:30.757+0800	done dumping config.locks (10 documents)
  2018-09-07T00:48:30.757+0800	writing config.lockpings to 
  2018-09-07T00:48:30.786+0800	done dumping config.lockpings (10 documents)
  2018-09-07T00:48:30.786+0800	writing config.changelog to 
  2018-09-07T00:48:30.876+0800	done dumping db2.tb1 (9999 documents)
  2018-09-07T00:48:30.876+0800	writing config.collections to 
  2018-09-07T00:48:30.881+0800	done dumping db2.table1 (6526 documents)
  2018-09-07T00:48:30.881+0800	writing config.chunks to 
  2018-09-07T00:48:30.882+0800	done dumping config.changelog (9 documents)
  2018-09-07T00:48:30.882+0800	writing config.shards to 
  2018-09-07T00:48:30.889+0800	done dumping config.shards (3 documents)
  2018-09-07T00:48:30.889+0800	writing config.databases to 
  2018-09-07T00:48:30.889+0800	done dumping config.collections (3 documents)
  2018-09-07T00:48:30.889+0800	writing config.version to 
  2018-09-07T00:48:30.891+0800	done dumping config.chunks (3 documents)
  2018-09-07T00:48:30.891+0800	writing config.mongos to 
  2018-09-07T00:48:30.921+0800	done dumping testdb.table1 (9999 documents)
  2018-09-07T00:48:30.921+0800	writing config.migrations to 
  2018-09-07T00:48:30.922+0800	done dumping config.version (1 document)
  2018-09-07T00:48:30.922+0800	writing config.tags to 
  2018-09-07T00:48:30.923+0800	done dumping config.databases (2 documents)
  2018-09-07T00:48:30.923+0800	done dumping config.mongos (1 document)
  2018-09-07T00:48:30.925+0800	done dumping config.migrations (0 documents)
  2018-09-07T00:48:30.926+0800	done dumping config.tags (0 documents)
  
  $ ls /tmp/mongobak/
  admin  config  db2  testdb
  
  ```

- 备份指定集合：

  ```bash
  mongodump --host [ip] --port [port] -d [db] -c [collection] -o [/path/dir/]
  ```

  ```bash
  $ mongodump --port 20000 -d db2 -c tb1 -o /tmp/db2
  2018-09-07T00:52:31.596+0800	writing db2.tb1 to 
  2018-09-07T00:52:31.645+0800	done dumping db2.tb1 (9999 documents)
  $ ls /tmp/db2/
  db2
  $ ls /tmp/db2/db2/
  tb1.bson  tb1.metadata.json 
  
  ```

- 导出集合为json文件：

  ```bash
  mongoexport --host [ip] --port [port] -d [db] -c [collection] -o [/path/dir/name.json]
  ```

  ```bash
  $ mongoexport --port 20000 -d db2 -c tb1 -o /tmp/db2/tb1.json
  2018-09-07T00:54:42.632+0800	connected to: localhost:20000
  2018-09-07T00:54:42.933+0800	exported 9999 records
  
  # json格式数据
  $ cat /tmp/db2/tb1.json 
  {"_id":{"$oid":"5b915896464b3a14f3e21883"},"id":1.0,"t2":"testval2"}
  {"_id":{"$oid":"5b915896464b3a14f3e21884"},"id":2.0,"t2":"testval2"}
  {"_id":{"$oid":"5b915896464b3a14f3e21885"},"id":3.0,"t2":"testval2"}
  {"_id":{"$oid":"5b915896464b3a14f3e21886"},"id":4.0,"t2":"testval2"}
  {"_id":{"$oid":"5b915896464b3a14f3e21887"},"id":5.0,"t2":"testval2"}
  ...
  ```

## 恢复

- 恢复所有库：

  ```bash
  mongorestore --host [ip] --port [port] --drop /path/dir/
  ```

  - 这里dir是备份所有库的目录的名字，其中--drop为可选参数，作用是在恢复之前先将数据情况，不建议使用。

  ```bash
  # 删除数据库
  mongos> show dbs
  admin   0.000GB
  config  0.001GB
  db2     0.001GB
  testdb  0.000GB
  mongos> db.drop
  db.dropAllRoles(  db.dropAllUsers(  db.dropDatabase(  db.dropRole(      db.dropUser(
  mongos> db.dropDatabase(db2)
  2018-09-07T01:02:50.898+0800 E QUERY    [thread1] ReferenceError: db2 is not defined :
  @(shell):1:1
  mongos> use db2
  switched to db db2
  mongos> db.dropDatabase()
  {
  	"dropped" : "db2",
  	"ok" : 1,
  	"operationTime" : Timestamp(1536253396, 11),
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536253396, 11),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  mongos> show dbs
  admin   0.000GB
  config  0.001GB
  testdb  0.000GB
  mongos> use testdb
  switched to db testdb
  mongos> db.dropDatabase()
  {
  	"dropped" : "testdb",
  	"ok" : 1,
  	"operationTime" : Timestamp(1536253408, 18),
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536253408, 18),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  
  # 报错是由于config和admin数据库无法恢复，需要先删除
  $ mongorestore --port 20000 --drop /tmp/mongobak/
  2018-09-07T01:04:55.684+0800	preparing collections to restore from
  2018-09-07T01:04:55.685+0800	Failed: cannot do a full restore on a sharded system - remove the 'config' directory from the dump directory first
  
  $ rm -rf /tmp/mongobak/config/
  $ mongorestore --port 20000 --drop /tmp/mongobak/
  2018-09-07T01:06:01.752+0800	preparing collections to restore from
  2018-09-07T01:06:01.756+0800	reading metadata for testdb.table1 from /tmp/mongobak/testdb/table1.metadata.json
  2018-09-07T01:06:01.769+0800	reading metadata for db2.tb1 from /tmp/mongobak/db2/tb1.metadata.json
  2018-09-07T01:06:01.790+0800	reading metadata for db2.table1 from /tmp/mongobak/db2/table1.metadata.json
  2018-09-07T01:06:02.399+0800	restoring testdb.table1 from /tmp/mongobak/testdb/table1.bson
  2018-09-07T01:06:02.423+0800	restoring db2.tb1 from /tmp/mongobak/db2/tb1.bson
  2018-09-07T01:06:02.492+0800	restoring db2.table1 from /tmp/mongobak/db2/table1.bson
  2018-09-07T01:06:03.231+0800	no indexes to restore
  2018-09-07T01:06:03.231+0800	finished restoring db2.table1 (6526 documents)
  2018-09-07T01:06:03.393+0800	restoring indexes for collection testdb.table1 from metadata
  2018-09-07T01:06:03.396+0800	restoring indexes for collection db2.tb1 from metadata
  2018-09-07T01:06:03.488+0800	finished restoring testdb.table1 (9999 documents)
  2018-09-07T01:06:03.498+0800	finished restoring db2.tb1 (9999 documents)
  2018-09-07T01:06:03.498+0800	done
  
  # 恢复成功
  mongos> show dbs
  admin   0.000GB
  config  0.001GB
  db2     0.000GB
  testdb  0.000GB
  ```

- 恢复指定库：

  ```bash
  mongorestore --host [ip] --port [port] -d [db] /bak/dir/
  ```

  - 其中-d指定恢复库的名字，dir为该库备份时所在目录

  ```bash
  $ mongorestore --port 20000 -d db2 /tmp/db2/db2/
  2018-09-07T01:10:43.043+0800	the --db and --collection args should only be used when restoring from a BSON file. Other uses are deprecated and will not exist in the future; use --nsInclude instead
  2018-09-07T01:10:43.043+0800	building a list of collections to restore from /tmp/db2/db2 dir
  2018-09-07T01:10:43.046+0800	reading metadata for db2.tb1 from /tmp/db2/db2/tb1.metadata.json
  2018-09-07T01:10:43.309+0800	restoring db2.tb1 from /tmp/db2/db2/tb1.bson
  2018-09-07T01:10:43.811+0800	restoring indexes for collection db2.tb1 from metadata
  2018-09-07T01:10:43.843+0800	finished restoring db2.tb1 (9999 documents)
  2018-09-07T01:10:43.843+0800	done
  
  ```

- 恢复集合：

  ```bash
  mongorestore --host [ip] --port [port] -d [db] -c [collection] /dir/db/collection.bson
  ```

  - 其中-c指定要恢复的集合的名字，dir为备份时生成文件所在路径，这里要指定到集合的bson文件路径。

  ```bash
  # 删除集合
  mongos> use db2
  switched to db db2
  mongos> show tables
  tb1
  mongos> db.tb1.drop()
  true
  mongos> show tables;
  mongos> 
  bye
  
  $ mongorestore --port 20000 -d db2 -c tb1 /tmp/db2/db2/tb1.bson 
  2018-09-07T01:19:28.287+0800	checking for collection data in /tmp/db2/db2/tb1.bson
  2018-09-07T01:19:28.289+0800	reading metadata for db2.tb1 from /tmp/db2/db2/tb1.metadata.json
  2018-09-07T01:19:28.297+0800	restoring db2.tb1 from /tmp/db2/db2/tb1.bson
  2018-09-07T01:19:28.559+0800	restoring indexes for collection db2.tb1 from metadata
  2018-09-07T01:19:28.588+0800	finished restoring db2.tb1 (9999 documents)
  2018-09-07T01:19:28.588+0800	done
  
  ```

- 导入集合：

  ```bash
  mongodimport --host [ip] --port [port] -d [db] -c [collection] --file /dir/collection.json
  ```

  ```bash
  $ mongoimport --port 20000 -d db2 -c tb1 --file /tmp/db2/tb1.json 
  2018-09-07T01:23:38.029+0800	connected to: localhost:20000
  2018-09-07T01:23:38.598+0800	imported 9999 documents
  
  ```

---