---
title: 使用beats采集日志
author: Evobot
date: 2020-04-26 21:00:19
categories: ELK
tags: beats
image:
---





<!--more-->

---

# 使用Beats采集日志

## 安装filebeat及配置

- logstash采集日志占用着资源较高，而Beats属于轻量的日志采集器；

- Beats集合了多种单一用途的数据采集器，例如filebeat，auditbeat，winlogbeat等；

- 这里我们下载filebeat在133节点上采集日志，filebeat是针对日志的数据采集器，下载filebeat的[RPM安装包](https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.0.0-x86_64.rpm)进行安装；

- 安装完成后，编辑filebeat的配置文件`/etc/filebeat/filebeat.yml`，修改配置文件，找到paths选项，注释掉enable选项，并修改下面的log文件路径：

  ```yaml
  - type: log
  
    # Change to true to enable this prospector configuration.
   # enabled: false 注释改行或者改为true
  
    # Paths that should be crawled and fetched. Glob based paths.
    paths:
      - /var/log/messages
      #- c:\programdata\elasticsearch\logs\*
  
  ```

- 然后注释掉`output.elasticsearch`该项配置，增加`output.console`：

  ```yaml
  output.console:
    enable: true
  ```

- 保存退出，使用下面的命令指定配置文件前台启动filebeat：

  ```bash
  /usr/share/filebeat/bin/filebeat -c /etc/filebeat/filebeat.yml
  ```

- 正常启动后，前台就会打印收集到的syslog日志。

## filebeat采集日志到elasticsearch

- 配置filebeat采集es的日志并传送到elasticsearch；

- 修改配置文件中的paths，日志路径指向es的日志：

  ```yaml
  filebeat.prospectors:
  
  # Each - is a prospector. Most options can be set at the prospector level, so
  # you can use different prospectors for various configurations.
  # Below are the prospector specific configurations.
  
  - type: log
  
    # Change to true to enable this prospector configuration.
    enabled: true
  
    # Paths that should be crawled and fetched. Glob based paths.
    paths:
      - /var/log/elasticsearch/evobot_es_test.log
  
  ```

- 然后删除上面的`output.console`配置，去掉`output.elasticsearch`配置的注释，修改hosts的ip为主节点IP：

  ```yaml
  output.elasticsearch:
    # Array of hosts to connect to.
    hosts: ["192.168.139.128:9200"]
  
  ```

- 保存退出，使用`systemctl start filebeat`启动filebeat服务；

- 然后到es主节点上查看是否生成了filebeat的索引：

  ```bash
  [root@centos_1 ~]# curl '192.168.139.128:9200/_cat/indices?v'
  health status index                     uuid                   pri rep docs.count docs.deleted store.size pri.store.size
  green  open   filebeat-6.0.0-2020.04.26 H71T76_VRCq1lNF8BR3Qyw   3   1       6637            0      1.4mb        716.7kb
  green  open   nginx-test-2020.04.22     AFBLvC5ZSAqMCYYf0CmcZg   5   1         52            0    937.4kb        468.7kb
  green  open   nginx-test-2020.04.26     7HmAgLByRAOlwwPXkjtO_g   5   1       9189            0      2.2mb          1.2mb
  green  open   system-syslog-2020.04     RxYvkltySdmeHGbTWlr-Zg   5   1      24562            0      5.4mb          2.6mb
  green  open   .kibana                   OpdMWp6vTra_HhymSyagOQ   1   1          3            1     40.3kb         20.1kb
  
  ```

  可以看到生成了自动命名的filebeat索引。

## kibana展示filebeat采集的日志

- 访问kibana，添加filebeat的索引：

  ![kibana-filebeat](https://s1.ax1x.com/2020/04/26/J27QbT.png)

- 添加成功后，进入Discover菜单，选中filebeat索引，便能看到采集的日志。