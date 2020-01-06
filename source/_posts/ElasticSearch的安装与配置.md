---
title: ElasticSearch的安装与配置
author: Evobot
categories: ELK
tags: ElasticSearch
abbrlink: c70364b4
date: 2019-06-12 22:43:47
image:
---

1. 安装ElasticSearch
2. 配置ElasticSearch
3. curl查看ElasticSearch

<!--more-->

---

#  安装ElasticSearch

- 安装ElasticSearch可以参考[官方文档](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html);

- 实际安装ElasticSearch可以使用yum安装，首先需要导入rpm的key：

  ```bash
  rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
  ```

- 然后在`/etc/yum.repos.d/`目录下创建elasticsearch的yum仓库文件`elastic.repo`，写入以下内容：

  ```ini
  [elasticsearch-6.x]
  name=Elasticsearch repository for 6.x packages
  baseurl=https://artifacts.elastic.co/packages/6.x/yum
  gpgcheck=1
  gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
  enabled=1
  autorefresh=1
  type=rpm-md
  
  ```

- 完成后就可以使用命令`yum list | grep elastic`查看仓库是否生效：

  ```bash
  [root@centos_1 ~]# yum list |grep elastic
  apm-server.i686                             6.8.0-1                    elasticsearch-6.x
  apm-server.x86_64                           6.8.0-1                    elasticsearch-6.x
  auditbeat.i686                              6.8.0-1                    elasticsearch-6.x
  auditbeat.x86_64                            6.8.0-1                    elasticsearch-6.x
  elasticsearch.noarch                        6.8.0-1                    elasticsearch-6.x
  filebeat.i686                               6.8.0-1                    elasticsearch-6.x
  filebeat.x86_64                             6.8.0-1                    elasticsearch-6.x
  heartbeat-elastic.i686                      6.8.0-1                    elasticsearch-6.x
  heartbeat-elastic.x86_64                    6.8.0-1                    elasticsearch-6.x
  journalbeat.i686                            6.8.0-1                    elasticsearch-6.x
  journalbeat.x86_64                          6.8.0-1                    elasticsearch-6.x
  kibana.x86_64                               6.8.0-1                    elasticsearch-6.x
  kibana-oss.x86_64                           6.3.0-1                    elasticsearch-6.x
  logstash.noarch                             1:6.8.0-1                  elasticsearch-6.x
  metricbeat.i686                             6.8.0-1                    elasticsearch-6.x
  metricbeat.x86_64                           6.8.0-1                    elasticsearch-6.x
  packetbeat.i686                             6.8.0-1                    elasticsearch-6.x
  packetbeat.x86_64                           6.8.0-1                    elasticsearch-6.x
  pcp-pmda-elasticsearch.x86_64               4.1.0-5.el7_6              updates
  rsyslog-elasticsearch.x86_64                8.24.0-34.el7              base
  
  ```

- 仓库生效后，使用`yum install -y elasticsearch`安装ElasticSearch，安装速度会比较慢，可以到[官方地址](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.0.0.rpm)直接下载rpm包进行安装，这里提供的rpm包地址为6.0.0的版本。

- 对于下载的rpm包，使用`rpm -ivh elasticsearch-6.0.0.rpm`命令进行安装：

  ```bash
  [root@centos_1 ~]# rpm -ivh elasticsearch-6.0.0.rpm
  准备中...                          ################################# [100%]
  Creating elasticsearch group... OK
  Creating elasticsearch user... OK
  正在升级/安装...
     1:elasticsearch-0:6.0.0-1          ################################# [100%]
  ### NOT starting on installation, please execute the following statements to configure elasticsearch service to start automatically using systemd
   sudo systemctl daemon-reload
   sudo systemctl enable elasticsearch.service
  ### You can start elasticsearch service by executing
   sudo systemctl start elasticsearch.service
  
  ```

- 在三台机器上都安装elasticsearch软件包。

# 配置ElasticSearch

- 安装完elasticsearch后，可以使用`rpm -ql elasticsearch`查看es软件包安装的文件：

  ```bash
  [root@centos_1 ~]# rpm -ql elasticsearch
  /etc/elasticsearch/elasticsearch.yml
  /etc/elasticsearch/jvm.options
  /etc/elasticsearch/log4j2.properties
  /etc/init.d/elasticsearch
  /etc/sysconfig/elasticsearch
  /usr/lib/sysctl.d/elasticsearch.conf
  /usr/lib/systemd/system/elasticsearch.service
  /usr/lib/tmpfiles.d/elasticsearch.conf
  ...
  ```

- 其中es有两个配置文件，分别为`/etc/elasticsearch/elasticsearch.yml`和`/etc/sysconfig/elasticsearch`，elasticsearch配置文件是es服务本身的配置文件，如配置文件PATH，PID，JAVA相关选项等等，而elasticsearch.yml则是集群、节点、数据、日志路径、监听IP和端口等的配置文件；

- 首先在主节点机器128上编辑`/etc/elasticsearch/elasticsearch.yml`文件，增加或更改以下配置：

  ```yaml
  cluster.name: evobot_es_test
  node.name: vm1
  node.master: true //表示该机器为主节点
  node.data: false  //表示该机器非数据节点
  #network.host: 0.0.0.0 //监听全部IP，但不够安全，在机器有公网IP的情况下，建议指定监听内网IP
  network.host: 192.168.139.128
  http.port: 9200
  discovery.zen.ping.unicast.hosts: ["centos_1", "centos_2", "centos_3"]  //定义集群内的角色，可以使用hosts定义过的主机名或者IP地址
  ```

- 编辑完之后保存，然后在另外两台数据节点同样编辑配置文件，并根据实际角色修改相应的配置，如下：

  ```yaml
  cluster.name: evobot_es_test
  node.name: vm2
  node.master: false
  node.data: true
  network.host: 192.168.139.130
  http.port: 9200
  discovery.zen.ping.unicast.hosts: ["centos_1", "centos_2", "centos_3"]
  ```

- 三台机器都配置完成后，就可以启动三台机器的es服务，使用`systemctl start elasticsearch`命令启动服务，启动后查看es进程是否正常运行：

  ```bash
  [root@centos_1 ~]# systemctl start elasticsearch
  [root@centos_1 ~]# ps aux |grep elasticsearch
  elastic+  10924 59.5 44.1 3510316 822924 ?      Ssl  23:58   0:04 /bin/java -Xms1g -Xmx1g -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -XX:+AlwaysPreTouch -server -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/var/lib/elasticsearch -Des.path.home=/usr/share/elasticsearch -Des.path.conf=/etc/elasticsearch -cp /usr/share/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch -p /var/run/elasticsearch/elasticsearch.pid --quiet
  root      10967  0.0  0.0 112724   984 pts/0    S+   23:58   0:00 grep --color=auto elasticsearch
  
  ```

- 如果es进程没有正常运行，首先查看`/var/log/elasticsearch/`目录下是否有日志，如果该目录下没有日志，说明不是es的配置问题导致的无法启动，再去查看`/var/log/message`中的日志信息：

  ```bash
  Jun 18 23:59:28 vm2 elasticsearch: which: no java in (/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin)
  Jun 18 23:59:28 vm2 systemd: elasticsearch.service: main process exited, code=exited, status=1/FAILURE
  Jun 18 23:59:28 vm2 elasticsearch: could not find java; set JAVA_HOME or ensure java is in PATH
  
  ```

  根据以上日志可以判断，系统的JDK环境存在问题导致es无法启动，需要查看JAVA_HOME的配置是否正确，首先查看java是否安装，然后/etc/profile内的java环境变量是否添加了export，再使用命令`ln -s /usr/local/jdk1.8/bin/java /usr/bin/`将java命令软连接到/usr/bin下。

- 服务启动后，查看`/var/log/elasticsearch/[cluster_name].log`是否存在报错，例如：

  ```bash
  [2019-06-19T00:10:14,193][INFO ][o.e.d.z.ZenDiscovery     ] [vm2] failed to send join request to master [{vm1}{Gi8jOhmWQzC5c1irLNrMTw}{dPNVNuNsSA26h0Fb4khxbw}{192.168.139.128}{192.168.139.128:9300}], reason [RemoteTransportException[[vm1][192.168.139.128:9300][internal:discovery/zen/join]]; nested: ConnectTransportException[[vm2][192.168.139.130:9300] connect_timeout[30s]]; nested: IOException[没有到主机的路由: 192.168.139.130/192.168.139.130:9300]; nested: IOException[没有到主机的路由]; ]
  
  ```

  上面的报错是由于selinux没有关闭或者防火墙规则导致，关闭selinux并在防火墙添加允许es的9200和9300端口后，重启es服务，其中9200端口是集群本身使用的端口，9300则是节点数据传输使用的端口。

# curl查看ElasticSearch

- 检测es服务是否健康，可以使用`curl '192.168.139.128:9200/_cluster/health?pretty'`命令对主节点的es服务健康状况进行检测：

  ```bash
  [root@centos_1 ~]# curl '192.168.139.128:9200/_cluster/health?pretty'
  {
    "cluster_name" : "evobot_es_test",
    "status" : "green",	#健康
    "timed_out" : false,  #无超时
    "number_of_nodes" : 3, #3个节点
    "number_of_data_nodes" : 2, #两个数据节点
    "active_primary_shards" : 0,
    "active_shards" : 0,
    "relocating_shards" : 0,
    "initializing_shards" : 0,
    "unassigned_shards" : 0,
    "delayed_unassigned_shards" : 0,
    "number_of_pending_tasks" : 0,
    "number_of_in_flight_fetch" : 0,
    "task_max_waiting_in_queue_millis" : 0,
    "active_shards_percent_as_number" : 100.0
  }
  
  ```

- 除了查看es的健康信息，还可以查看es的集群信息，使用`curl '192.168.139.128:9200/_cluster/state?pretty'`命令进行查看：

  ```bash
  [root@centos_1 ~]# curl '192.168.139.128:9200/_cluster/state?pretty'
  {
    "cluster_name" : "evobot_es_test",
    "compressed_size_in_bytes" : 348,
    "version" : 6,
    "state_uuid" : "G2qRbuQ3Tge5n2SDghXCeg",
    "master_node" : "Gi8jOhmWQzC5c1irLNrMTw",
    "blocks" : { },
    "nodes" : {
      "SQv2ouFRRMWKcWD-gZ6uIw" : {
        "name" : "vm3",
        "ephemeral_id" : "K3jxOnTWTBSXnm92d-Hrhg",
        "transport_address" : "192.168.139.131:9300",
        "attributes" : { }
      },
      "4jK1ltlJSM-gwC0RWY3rDg" : {
        "name" : "vm2",
        "ephemeral_id" : "l61iVh8tQlKNitSg6lyYCw",
        "transport_address" : "192.168.139.130:9300",
        "attributes" : { }
      },
      "Gi8jOhmWQzC5c1irLNrMTw" : {
        "name" : "vm1",
        "ephemeral_id" : "PIo1OF9HQSKNFxG70Puz-Q",
        "transport_address" : "192.168.139.128:9300",
        "attributes" : { }
      }
    },
  ...
  ```

---