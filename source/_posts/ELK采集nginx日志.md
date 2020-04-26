---
title: ELK采集nginx日志
author: Evobot
categories: ELK
tags:
  - logstash
  - kibana
abbrlink: 5ae97a45
date: 2020-04-22 21:34:17
image:
---



主要介绍如何使用logstash采集nginx的访问日志，并对logstash的配置文件格式有进一步的了解，实现在kibana上查看logstash采集到的nginx访问日志。

<!--more-->

---

# logstash采集nginx日志

## 创建nginx日志采集配置

- 在132数据节点上，创建收集nginx日志的配置文件；

- 创建并编辑`/etc/logstash/conf.d/nginx.conf`，写入以下配置：

  ```
  input {
    file {
      path => "/tmp/elk_access.log"
      start_position => "beginning"
      type => "nginx"
    }
  }
  filter {
    grok {
        match => { "message" => "%{IPORHOST:http_host} %{IPORHOST:clientip} - %{USERNAME:remote_user} \[%{HTTPDATE:timestamp}\]\"(?:%{WORD:http_verb} %{NOTSPACE:http_request}(?: HTTP/%{NUMBER:http_version})?|%{DATA:raw_http_request})\" %{NUMBER:response} (?:%{NUMBER:bytes_read}|-) %{QS:referrer} %{QS:agent} %{QS:xforwardedfor} %{NUMBER:request_time:float}"}
    }
    geoip {
        source => "clientip"
    }
  }
  output {
      stdout { codec => rubydebug }
      elasticsearch {
          hosts => ["192.168.139.132:9200"]
          index => "nginx-test-%{+YYYY.MM.dd}"
      }
  }
  
  ```

- 这里定义了input日志源来自指定的文件，然后start_position指定了日志开始记录的位置，type则是自定义的类型，filter是对日志的格式进行过滤，指定了filter的gork，gork是数据结构化转换工具， 采用组合多个预定义的正则表达式，用来匹配分割文本并映射到关键字的工具 ，相应的nginx的访问日志格式也要更改，output则是输出的对象，指定了屏幕打印和输出到elasticsearch；

- 编辑完成后保存退出，执行对配置文件进行检查：

  ```bash
  [root@centos2 ~]# cd /usr/share/logstash/bin/
  [root@centos2 bin]# ./logstash --path.settings /etc/logstash/ -f /etc/logstash/conf.d/nginx.conf --config.test_and_exit
  Sending Logstash's logs to /var/log/logstash which is now configured via log4j2.properties
  Configuration OK
  
  ```

## nginx配置

- 在132数据节点的nginx上增加一个虚拟主机配置如下：

  ```bash
  server {
          listen 80;
          server_name elk.evobot.cn;
  
          location / {
              proxy_pass http://192.168.139.128:5601;
              proxy_set_header Host $host;
              proxy_set_header X-Real-IP  $remote_addr;
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          }
          access_log      /tmp/elk_access.log main2;
  }
  
  ```

  这里的配置实际上是反向代理kibana，同时将nginx的访问日志配置成和logstash里配置的相同。

- 上面nginx的日志指定了main2日志格式，所以还需要再nginx的主配置文件中增加main2日志文件格式配置，编辑`/usr/local/nginx/conf/nginx.conf`，找到`log_format`配置项，在下面增加一行，写入以下配置：

  ```
  log_format main2 '$http_host $remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" "$upstream_addr" $request_time';
  
  ```

- 配置nginx完成后，检测nginx配置文件：

  ```bash
  [root@centos2 vhosts]# /usr/local/nginx/sbin/nginx -t
  nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
  nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
  
  ```

- 启动nginx，配置hosts指向nginx虚拟主机中配置的域名，然后就可以访问到代理的kibana，同时查看访问日志是否生成：

  ```bash
  [root@centos2 vhosts]# ls -l /tmp/elk_access.log
  -rw-r--r-- 1 root root 8025 4月  22 22:39 /tmp/elk_access.log
  [root@centos2 vhosts]# tail /tmp/elk_access.log
  elk.evobot.cn 192.168.139.1 - - [22/Apr/2020:22:39:35 +0800] "GET /plugins/kibana/assets/wrench.svg HTTP/1.1" 200 290 "http://elk.evobot.cn/app/kibana" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.113 Safari/537.36" "192.168.139.128:5601" 0.028
  elk.evobot.cn 192.168.139.1 - - [22/Apr/2020:22:39:35 +0800] "GET /plugins/kibana/assets/settings.svg HTTP/1.1" 200 308 "http://elk.evobot.cn/app/kibana" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.113 Safari/537.36" "192.168.139.128:5601" 0.018
  elk.evobot.cn 192.168.139.1 - - [22/Apr/2020:22:39:35 +0800] "GET /bundles/ae11252ad19209059498cac1cd1addd7.svg HTTP/1.1" 2001188 "http://elk.evobot.cn/bundles/commons.style.css?v=16070" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/81.0.4044.113 Safari/537.36" "192.168.139.128:5601" 0.015
  
  ```

## kibana配置

- 完成nginx配置后，执行`systemctl restart logstash`重启logstash的服务，开始收集nginx的日志；

- 重启成功后，到主节点128上查看是否生成索引：

  ```bash
  [root@centos_1 ~]# curl '192.168.139.128:9200/_cat/indices?v'
  health status index                 uuid                   pri rep docs.count docs.deleted store.size pri.store.size
  green  open   nginx-test-2020.04.22 AFBLvC5ZSAqMCYYf0CmcZg   5   1         27            0    200.8kb        100.4kb
  green  open   system-syslog-2020.04 RxYvkltySdmeHGbTWlr-Zg   5   1        189            0    702.7kb        368.9kb
  green  open   .kibana               OpdMWp6vTra_HhymSyagOQ   1   1          2            1     27.2kb         13.6kb
  
  ```

  可以看到已经生成了nginx的索引。

- 访问kibana，点击左侧`Management`菜单，在右侧点击`Index Patterns`，然后在左侧`system-syslog-*`上方点击`Create Index Pattern`创建新的索引：

  ![kibana-nginx](https://s1.ax1x.com/2020/04/22/JUYzwQ.png)

- 填入查看到的nginx的索引号`nginx-test-*`，然后点击Create。

- 回到Discover菜单，选择nginx的索引，就可以看到es采集到的nginx访问日志：

  ![kibana-nginx2](https://s1.ax1x.com/2020/04/22/JUtB1P.png)

---

