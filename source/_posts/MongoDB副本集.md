---
title: MongoDB副本集
author: Evobot
date: 2018-09-05 21:30:29
categories: NoSQL
tags:
  - MongoDB
image:
---

1. MongoDB副本集介绍
2. MongoDB副本集搭建
3. MongoDB副本集测试

<!--more-->

---

# MongoDB副本集介绍

- 早期MongoDB高可用使用的是master-slave架构，一主一从与MySQL类似，但在此架构中slave为只读模式，当主库宕机后，从库不能自动切换为主；

- 目前版本的MongoDB则采用的是副本集架构的高可用，这种模式下有一个主（primary），以及多个从（secondary），其中从是只读的，并且支持设置权重，当主宕机后，权重最高的从切换为主；

- 另外在副本集架构中还可以建立一个仲裁（arbiter）的角色，它只负责裁决主库是否宕机，而不存储数据；

- 副本集架构中，读写数据都是在主上，要想实现负载均衡的目的，则需要手动指定读库的目标server；

- 副本集架构图

  [![iSMgAA.md.png](https://s1.ax1x.com/2018/09/03/iSMgAA.md.png)](https://imgchr.com/i/iSMgAA)

# MongoDB副本集搭建

- 首先在准备三台主机，IP分别为192.168.49.128，192.168.49.129,192.168.49.130，三台主机均安装MongoDB，并关闭selinux和防火墙，其中128为主机，另外两台为从机；

- 修改MongoDB的配置文件`/etc/mongod.conf`，将`bindIp`监听地址增加本地内网地址，并且取消`replcation`选项的注释，三台机器的实际配置相同，如下：

  ```bash
  net:
    port: 27017
    bindIp: 127.0.0.1,192.168.49.130
  
  replication:
    oplogSizeMB: 20
    replSetName: evobot
  
  ```

  - `oplogSizeMB`是定义oplog日志的大小，可以不配置；
  - `replSetName`则是定义副本集的名字，三台机器的配置相同。

- 然后连接主的MongoDB命令行，执行下面的命令：

  ```json
  > use admin
  switched to db admin
  > config={_id:"evobot",members:[{_id:0,host:"192.168.49.128:27017"},{_id:1,host:"192.168.49.129"},{_id:2,host:"192.168.49.130"}]}
  
  ```

  - id:0是主，剩下两个为从

- 然后执行`rs.initiate(config)`初始化命令：

  ```json
  > rs.initiate(config)
  {
  	"ok" : 1,
  	"operationTime" : Timestamp(1536155549, 1),
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536155549, 1),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  
  ```

  - 显示`"ok": 1`则表示副本集创建成功。

- `rs.status()`命令可以查看副本集的状态：

  ```json
  evobot:SECONDARY> rs.status()
  {
  	"set" : "evobot",
  	"date" : ISODate("2018-09-05T13:53:06.737Z"),
  	"myState" : 1,
  	"term" : NumberLong(1),
  	"syncingTo" : "",
  	"syncSourceHost" : "",
  	"syncSourceId" : -1,
  	"heartbeatIntervalMillis" : NumberLong(2000),
  	"optimes" : {
  		"lastCommittedOpTime" : {
  			"ts" : Timestamp(1536155581, 1),
  			"t" : NumberLong(1)
  		},
  		"readConcernMajorityOpTime" : {
  			"ts" : Timestamp(1536155581, 1),
  			"t" : NumberLong(1)
  		},
  		"appliedOpTime" : {
  			"ts" : Timestamp(1536155581, 1),
  			"t" : NumberLong(1)
  		},
  		"durableOpTime" : {
  			"ts" : Timestamp(1536155581, 1),
  			"t" : NumberLong(1)
  		}
  	},
  	"members" : [
  		{
  			"_id" : 0,
  			"name" : "192.168.49.128:27017",
  			"health" : 1,
  			"state" : 1,
  			"stateStr" : "PRIMARY",
  			"uptime" : 2446,
  			"optime" : {
  				"ts" : Timestamp(1536155581, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDate" : ISODate("2018-09-05T13:53:01Z"),
  			"syncingTo" : "",
  			"syncSourceHost" : "",
  			"syncSourceId" : -1,
  			"infoMessage" : "could not find member to sync from",
  			"electionTime" : Timestamp(1536155559, 1),
  			"electionDate" : ISODate("2018-09-05T13:52:39Z"),
  			"configVersion" : 1,
  			"self" : true,
  			"lastHeartbeatMessage" : ""
  		},
  		{
  			"_id" : 1,
  			"name" : "192.168.49.129:27017",
  			"health" : 1,
  			"state" : 2,
  			"stateStr" : "SECONDARY",
  			"uptime" : 37,
  			"optime" : {
  				"ts" : Timestamp(1536155581, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDurable" : {
  				"ts" : Timestamp(1536155581, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDate" : ISODate("2018-09-05T13:53:01Z"),
  			"optimeDurableDate" : ISODate("2018-09-05T13:53:01Z"),
  			"lastHeartbeat" : ISODate("2018-09-05T13:53:05.606Z"),
  			"lastHeartbeatRecv" : ISODate("2018-09-05T13:53:06.026Z"),
  			"pingMs" : NumberLong(0),
  			"lastHeartbeatMessage" : "",
  			"syncingTo" : "192.168.49.128:27017",
  			"syncSourceHost" : "192.168.49.128:27017",
  			"syncSourceId" : 0,
  			"infoMessage" : "",
  			"configVersion" : 1
  		},
  		{
  			"_id" : 2,
  			"name" : "192.168.49.130:27017",
  			"health" : 1,
  			"state" : 2,
  			"stateStr" : "SECONDARY",
  			"uptime" : 37,
  			"optime" : {
  				"ts" : Timestamp(1536155581, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDurable" : {
  				"ts" : Timestamp(1536155581, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDate" : ISODate("2018-09-05T13:53:01Z"),
  			"optimeDurableDate" : ISODate("2018-09-05T13:53:01Z"),
  			"lastHeartbeat" : ISODate("2018-09-05T13:53:05.601Z"),
  			"lastHeartbeatRecv" : ISODate("2018-09-05T13:53:06.032Z"),
  			"pingMs" : NumberLong(0),
  			"lastHeartbeatMessage" : "",
  			"syncingTo" : "192.168.49.128:27017",
  			"syncSourceHost" : "192.168.49.128:27017",
  			"syncSourceId" : 0,
  			"infoMessage" : "",
  			"configVersion" : 1
  		}
  	],
  	"ok" : 1,
  	"operationTime" : Timestamp(1536155581, 1),
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536155581, 1),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  
  ```

  - 查看输出信息中的`members`，是否是一个PRIMARY，两个SECONDARY。

- 如果`rs.status()`中的members中，两个从的状态为`"stateStr":"STARTUP"`，则需要执行如下命令重新配置：

  ```json
  //重新配置config
  > var config={_id:"evobot",members:[{_id:0,host:"192.168.49.128:27017"},{_id:1,host:"192.168.49.129"},{_id:2,host:"192.168.49.130"}]}
  //重新初始化
  > rs.reconfig(config)
  ```

- 在副本集的创建过程中，想要哪一台机器作为主，就要在哪一台主机上执行配置副本集的命令。

# MongoDB副本集测试

## 副本集可用性测试

- 在主上创建库，并插入文档：

  ```json
  evobot:PRIMARY> use mydb
  switched to db mydb
  evobot:PRIMARY> db.acc.insert({AccountID:1,UserName:"123",password:"123456"})
  WriteResult({ "nInserted" : 1 })
  
  ```

- 查看创建的文档：

  ```json
  evobot:PRIMARY> show dbs
  admin   0.000GB
  config  0.000GB
  local   0.000GB
  mydb    0.000GB
  
  evobot:PRIMARY> use mydb
  switched to db mydb
  evobot:PRIMARY> show tables
  acc
  evobot:PRIMARY> db.acc.find()
  { "_id" : ObjectId("5b8fe273ef0e7acc8f910353"), "AccountID" : 1, "UserName" : "123", "password" : "123456" }
  
  ```

- 然后登录从机器的命令行，执行`show dbs`：

  ```json
  evobot:SECONDARY> show dbs
  2018-09-05T22:07:54.141+0800 E QUERY    [thread1] Error: listDatabases failed:{
  	"operationTime" : Timestamp(1536156471, 1),
  	"ok" : 0,
  	"errmsg" : "not master and slaveOk=false",
  	"code" : 13435,
  	"codeName" : "NotMasterNoSlaveOk",
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536156471, 1),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  } :
  _getErrorWithCode@src/mongo/shell/utils.js:25:13
  Mongo.prototype.getDBs@src/mongo/shell/mongo.js:65:1
  shellHelper.show@src/mongo/shell/utils.js:849:19
  shellHelper@src/mongo/shell/utils.js:739:15
  @(shellhelp2):1:1
  
  ```

  - 出现上面的报错`slaveOk=false`，说明从的状态不正确，此时需要执行`rs.slaveOk()`设置从：

  ```json
  evobot:SECONDARY> rs.slaveOk()
  evobot:SECONDARY> show dbs
  admin   0.000GB
  config  0.000GB
  local   0.000GB
  mydb    0.000GB
  evobot:SECONDARY> use mydb
  switched to db mydb
  evobot:SECONDARY> show tables
  acc
  evobot:SECONDARY> db.acc.find()
  { "_id" : ObjectId("5b8fe273ef0e7acc8f910353"), "AccountID" : 1, "UserName" : "123", "password" : "123456" }
  
  ```

- 同样的，在另一台从上也执行`rs.slaveOk()`。

## 更改权重模拟宕机

- 默认情况下，三台机器的权重都是1，使用`rs.config()`查看三台机器的权重`priority`：

  ```json
  evobot:PRIMARY> rs.config()
  {
  	"_id" : "evobot",
  	"version" : 1,
  	"protocolVersion" : NumberLong(1),
  	"members" : [
  		{
  			"_id" : 0,
  			"host" : "192.168.49.128:27017",
  			"arbiterOnly" : false,
  			"buildIndexes" : true,
  			"hidden" : false,
  			"priority" : 1,
  			"tags" : {
  				
  			},
  			"slaveDelay" : NumberLong(0),
  			"votes" : 1
  		},
  		{
  			"_id" : 1,
  			"host" : "192.168.49.129:27017",
  			"arbiterOnly" : false,
  			"buildIndexes" : true,
  			"hidden" : false,
  			"priority" : 1,
  			"tags" : {
  				
  			},
  			"slaveDelay" : NumberLong(0),
  			"votes" : 1
  		},
  		{
  			"_id" : 2,
  			"host" : "192.168.49.130:27017",
  			"arbiterOnly" : false,
  			"buildIndexes" : true,
  			"hidden" : false,
  			"priority" : 1,
  			"tags" : {
  				
  			},
  			"slaveDelay" : NumberLong(0),
  			"votes" : 1
  		}
  	],
  	"settings" : {
  		"chainingAllowed" : true,
  		"heartbeatIntervalMillis" : 2000,
  		"heartbeatTimeoutSecs" : 10,
  		"electionTimeoutMillis" : 10000,
  		"catchUpTimeoutMillis" : -1,
  		"catchUpTakeoverDelayMillis" : 30000,
  		"getLastErrorModes" : {
  			
  		},
  		"getLastErrorDefaults" : {
  			"w" : 1,
  			"wtimeout" : 0
  		},
  		"replicaSetId" : ObjectId("5b8fdf9cc66be9d311189b52")
  	}
  }
  
  ```

- 模拟宕机，在主上添加一条防火墙规则，然后在从上使用`rs.status()`查看副本集状态：

  ```bash
  iptables -I INPUT -p tcp --dport 27017 -j DROP
  ```

  ```json
  evobot:SECONDARY> rs.status()
  {
  	"set" : "evobot",
  	"date" : ISODate("2018-09-05T14:23:03.844Z"),
  	"myState" : 1,
  	"term" : NumberLong(2),
  	"syncingTo" : "",
  	"syncSourceHost" : "",
  	"syncSourceId" : -1,
  	"heartbeatIntervalMillis" : NumberLong(2000),
  	"optimes" : {
  		"lastCommittedOpTime" : {
  			"ts" : Timestamp(1536157361, 1),
  			"t" : NumberLong(1)
  		},
  		"readConcernMajorityOpTime" : {
  			"ts" : Timestamp(1536157361, 1),
  			"t" : NumberLong(1)
  		},
  		"appliedOpTime" : {
  			"ts" : Timestamp(1536157361, 1),
  			"t" : NumberLong(1)
  		},
  		"durableOpTime" : {
  			"ts" : Timestamp(1536157361, 1),
  			"t" : NumberLong(1)
  		}
  	},
  	"members" : [
  		{
  			"_id" : 0,
  			"name" : "192.168.49.128:27017",
  			"health" : 0,
  			"state" : 8,
  			"stateStr" : "(not reachable/healthy)", //可以看到主的状态已经异常
  			"uptime" : 0,
  			"optime" : {
  				"ts" : Timestamp(0, 0),
  				"t" : NumberLong(-1)
  			},
  			"optimeDurable" : {
  				"ts" : Timestamp(0, 0),
  				"t" : NumberLong(-1)
  			},
  			"optimeDate" : ISODate("1970-01-01T00:00:00Z"),
  			"optimeDurableDate" : ISODate("1970-01-01T00:00:00Z"),
  			"lastHeartbeat" : ISODate("2018-09-05T14:22:52.216Z"),
  			"lastHeartbeatRecv" : ISODate("2018-09-05T14:23:03.591Z"),
  			"pingMs" : NumberLong(0),
  			"lastHeartbeatMessage" : "Operation timed out",
  			"syncingTo" : "",
  			"syncSourceHost" : "",
  			"syncSourceId" : -1,
  			"infoMessage" : "",
  			"configVersion" : -1
  		},
  		{
  			"_id" : 1,
  			"name" : "192.168.49.129:27017",
  			"health" : 1,
  			"state" : 1,
  			"stateStr" : "PRIMARY", //从自动变成了主
  			"uptime" : 4160,
  			"optime" : {
  				"ts" : Timestamp(1536157361, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDate" : ISODate("2018-09-05T14:22:41Z"),
  			"syncingTo" : "",
  			"syncSourceHost" : "",
  			"syncSourceId" : -1,
  			"infoMessage" : "could not find member to sync from",
  			"electionTime" : Timestamp(1536157376, 1),
  			"electionDate" : ISODate("2018-09-05T14:22:56Z"),
  			"configVersion" : 1,
  			"self" : true,
  			"lastHeartbeatMessage" : ""
  		},
  		{
  			"_id" : 2,
  			"name" : "192.168.49.130:27017",
  			"health" : 1,
  			"state" : 2,
  			"stateStr" : "SECONDARY",
  			"uptime" : 1833,
  			"optime" : {
  				"ts" : Timestamp(1536157361, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDurable" : {
  				"ts" : Timestamp(1536157361, 1),
  				"t" : NumberLong(1)
  			},
  			"optimeDate" : ISODate("2018-09-05T14:22:41Z"),
  			"optimeDurableDate" : ISODate("2018-09-05T14:22:41Z"),
  			"lastHeartbeat" : ISODate("2018-09-05T14:23:02.474Z"),
  			"lastHeartbeatRecv" : ISODate("2018-09-05T14:23:03.834Z"),
  			"pingMs" : NumberLong(0),
  			"lastHeartbeatMessage" : "",
  			"syncingTo" : "192.168.49.128:27017",
  			"syncSourceHost" : "192.168.49.128:27017",
  			"syncSourceId" : 0,
  			"infoMessage" : "",
  			"configVersion" : 1
  		}
  	],
  	"ok" : 1,
  	"operationTime" : Timestamp(1536157361, 1),
  	"$clusterTime" : {
  		"clusterTime" : Timestamp(1536157376, 1),
  		"signature" : {
  			"hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
  			"keyId" : NumberLong(0)
  		}
  	}
  }
  
  ```

- 此时删除防火墙规则，模拟主宕机恢复，由于权重都为1，PRIMARY并不会回到原来的机器上；

- 为副本集配置权重，由于现在PRIMARY已经变到了192.168.49.129上，所以需要在新的PRIMARY的MongoDB命令行中进行如下操作，其中数字越大，权重越大：

  ```bash
  > cfg=rs.conf()
  > cfg.members[0].priority=3
  > cfg.members[1].priority=2
  > cfg.members[2].priority=1
  > rs.reconfig(cfg)
  
  ```

- 在执行了`rs.reconfig(cfg)`之后，由于主上的防火墙规则已经删除，会发现原本是`PRIMARY`的从，再次变为`SECONDARY`。