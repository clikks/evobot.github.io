---
title: MongoDB操作及php扩展
author: Evobot
categories: NoSQL
tags:
  - MongoDB
abbrlink: 84b729e8
date: 2018-08-27 23:07:29
image:
---

1. mongodb创建集合、数据管理
2. php的mongodb扩展
3. php的mongo扩展

<!--more-->

---

# mongodb创建集合和数据管理

- 在mongodb中创建集合，语法为`db.createCollection(name,options)`，例如：

  ```json
  db.createCollection("mycol",{capped:true, autoIndexId:true, size:6142800, max:10000})
  ```

  - 这里的mycol就是集合的名字，后面花括号内都是options，options是可选的，用来配置集合的参数；
  - `capped：true/false`表示是否启用封顶集合，封顶集合是固定大小的集合，当它达到最大大小，会自动覆盖最早的条目，如果capped选项为true，则后面需要指定集合的大小；
  - autoIndexId：表示是否自动创建索引_id字段，默认为false。
  - size：指定最大大小字节封顶集合，如果封顶为true，则需要指定此字段，单位为B；
  - max：指定封顶集合允许在文件的最大数量。
  - **需要注意，选项名是区分大小写的！**

- `show collections`或`show tables`可以查看当前数据库的集合：

  ```bash
  > show collections
  mycol
  > show tables;
  mycol

  ```

- `db.Account.insert({AccountID:1,UserName:"123",password:"123456"})`表示向Account集合中插入数据，如果集合不存在，mongodb会自动创建集合：

  ```json
  > db.Account.insert({AccountID:1,UserName:"123",password:"123456"})
  WriteResult({ "nInserted" : 1 })
  > show tables;
  Account
  mycol

  ```

- 更新集合中的记录，命令如下：

  ```json
  db.collection.update({AccountID:1},{"$set":{"Age":20}})
  ```

  ```json
  > db.Account.update({AccountID:1},{"$set":{"Age":20}})
  WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })

  ```

- `db.collection.find()`：用来查看集合中的文档：

  ```bash
  > db.Account.find()
  { "_id" : ObjectId("5b8422d141424a193a17ea11"), "AccountID" : 1, "UserName" : "123", "password" : "123456", "Age" : 20 }
  { "_id" : ObjectId("5b8422f641424a193a17ea12"), "AccountID" : 1, "UserName" : "evobot", "password" : "abcde" }

  ```

- `db.collection.find({condition})`：是根据条件condition查询集合中的文档：

  ```json
  > db.Account.find({"UserName":"evobot"})
  { "_id" : ObjectId("5b8422f641424a193a17ea12"), "AccountID" : 1, "UserName" : "evobot", "password" : "abcde" }

  ```

- `db.collection.remove({condition})`：是删除匹配condition条件的文档：

  ```bash
  > db.Account.remove({AccountID:1})
  WriteResult({ "nRemoved" : 2 })

  ```

- 删除集合使用`db.collection.drop()`：

  ```bash
  > show tables;
  Account
  > db.Account.drop()
  true
  > show tables;

  ```

- `db.printCollectionStats()`：用来查看集合的状态，会显示集合的大小、是否封顶的信息：

  ```bash
  > db.printCollectionStats()
  mycol2
  {
  	"ns" : "db1.mycol2",
  	"size" : 80,
  	"count" : 1,
  	"avgObjSize" : 80,
  	"storageSize" : 4096,
  	"capped" : false,
  	"wiredTiger" : {
  		"metadata" : {
  			"formatVersion" : 1
  ...
  ```

  ​


# PHP连接MongoDB

PHP支持mongodb有两个模块，一个是mongodb，一个是mongo，其中mongo只支持php5.x的版本，并且已经不再维护，建议使用mongodb模块，[官方文档](https://docs.mongodb.com/ecosystem/drivers/php/)可以查看两个模块的对比，以及下载地址。

## mongodb模块

- [下载](https://pecl.php.net/get/mongodb-1.5.2.tgz)最新的1.5.2版本的mongodb模块安装包;

- 解压安装包后，执行以下命令进行安装：

  ```bash
  $ /usr/local/php-fpm/bin/phpize
  $ ./configure --with-php-config=/usr/local/php-fpm/bin/php-config
  $ make && make install
  ```

- 安装成功后，在php.ini中添加`extension=mongodb.so`配置，重启php即可。

- 在nginx的默认虚拟主机中，创建测试php代码文件mongodb.php，写入以下内容：

  ```php
  <?php
  $bulk = new MongoDB\Driver\BulkWrite;
  $document = ['_id' => new MongoDB\BSON\ObjectID, 'name' => '菜鸟教程'];

  $_id= $bulk->insert($document);

  var_dump($_id);

  $manager = new MongoDB\Driver\Manager("mongodb://localhost:27017");  
  $writeConcern = new MongoDB\Driver\WriteConcern(MongoDB\Driver\WriteConcern::MAJORITY, 1000);
  $result = $manager->executeBulkWrite('test.runoob', $bulk, $writeConcern);
  ?>
  ```

- 然后使用`curl localhost/mongodb.php`访问代码文件，输入如下：

  ```bash
  $ curl localhost/mongodb.php
  object(MongoDB\BSON\ObjectId)#3 (1) {
    ["oid"]=>
    string(24) "5b88133bd69bb27ee07e7661"
  }

  ```

## mongo模块

- [下载](https://pecl.php.net/get/mongo-1.6.16.tgz)最新的1.6.16版本的mongo模块安装包；

- 安装及配置步骤与上面安装mongodb.so相同。

- 注意mongo模块只能用于php5.x。

- mongo的php测试脚本内容如下：

  ```php
  <?php
  $m = new MongoClient(); // 连接
  $db = $m->test; // 获取名称为 "test" 的数据库
  $collection = $db->createCollection("runoob");
  echo "集合创建成功";
  ?>
  ```

---

# 扩展

[mongodb安全设置]( http://www.mongoing.com/archives/631) 		[mongodb执行js脚本](http://www.jianshu.com/p/6bd8934bd1ca)

