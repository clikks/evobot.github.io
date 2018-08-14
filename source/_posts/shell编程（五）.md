---
title: shell编程（五）
author: Evobot
categories: shell编程
tags:
  - shell编程
abbrlink: 7a0f06ec
date: 2018-07-16 22:58:59
image:
---



本文详细列出监控系统的主脚本内容和配置文件的写法，以及监控系统负载，web请求502和磁盘空间三种监控脚本。

<!--more-->

---

# 告警系统主脚本

- 在`/usr/local/sbin/`下创建`mon`目录，作为监控系统脚本的根目录；

- 在`mon`目录下创建`bin`、`conf`、`shares`、`log`、`mail`目录；

- 在`bin`目录下创建`main.sh`文件，写入一下脚本内容：

  ```bash
  #!/bin/bash
  
  # 是否发送文件
  export send=1
  # 过滤ip地址
  export addr=`/sbin/ifconfig | grep -A1 'eth0' | grep inet | awk '{print$2}'`
  dir=`pwd`
  # 只需要最后一级目录
  last_dir=`echo $dir | awk -F'/' '{print $NF}'`
  
  # 下面的判断目的是保证执行脚本时，保证在bin目录里，否则可能找不到监控脚>本、邮件和日志
  if [ $last_dir == "bin" ] || [ $last_dir == "bin/" ]; then
      conf_file="../conf/mon.conf"
  else
      echo "you should cd bin dir"
      exit
  fi
  
  exec 1>>../log/mon.log 2>>../log/err.log
  echo "`date +"%F %T"` load_average"
  /bin/bash ../shares/load.sh
  
  # 检查配置文件中是否需要监控nginx的502错误
  if grep -q 'to_mon_502=1' $conf_file; then
      export log=`grep 'logfile'=' $conf_file | awk -F '=' '{print $2}' |sed 's/ //g'`
      /bin/bash ../shares/502.sh
  fi
  
  ```

  - `export`定义的变量在所有的子脚本中都可以使用。

# 告警系统配置文件

- 配置文件用来配置对应功能的开关，或者MySQL的ip、port、user/passwd等等；

- 在`conf`目录内，创建`mon.conf`文件，写入以下内容：

  ```bash
  ## to config the options if to monitor
  ## 定义MySQL的服务器地址、端口以及user、password
  to_mon_cdb=0 ## 0 or 1,default 0,0 not monitor, 1 monitor
  db_ip=10.20.3.13
  db_port=3315
  db_user=username
  db_pass=passwd
  
  ## httpd 1则监控，0则不监控
  to_mon_httpd=0
  
  ## php monitor
  to_mon_php_socket=0
  
  ## http_code_502 需要定义访问日志路径
  to_mon_502=1
  logfile=/data/log/xxx.com/access.log
  
  ## request_count 定义日志路径及域名
  to_mon_request_count=0
  req_log=/data/log/www.discuz.net/access.log
  # 需要获取请求数的域名
  domainname=www.discuz.net
  
  ```

# 告警系统监控项目

- 首先定义监控系统负载的脚本，在`shares`目录下，创建`load.sh`文件，写入以下内容：

  ```bash
  #!/bin/bash
  
  load=`uptime | awk -F 'average:' '{print $2}' | cut -d',' -f1 | sed 's/ //g' | cut -d'.' -f1'
  
  if [ $load -gt 10 ] && [ $send -eq "1" ]; then
      echo "$addr `date +%T` load is $load" > ../log/load.tmp
      /bin/bash ../mail/mail.sh $addr\_load $load ../log/load.tmp
  fi
  echo "`date +%T` load is $load"
  
  ```

- 继续创建`502.sh`文件，写入以下内容：

  ```bash
  #!/bin/bash
  
  d=`date -d "-1 min" +%H:%M`
  c_502=`grep :$d: $log |grep ' 502 ' | wc -l`
  
  if [ $c_502 -gt 10 ] && [ $send == 1 ]; then
      echo "$addr $d 502 count is $c_502" > ../log/502.tmp
      /bin/bash ../mail/mail.sh $addr\_502 $c_502 ../log/502.tmp
  fi
  
  echo "`date +%T` 502 $c_502"
  
  ```

- 增加监控磁盘使用率的监控脚本，创建`disk.sh`文件，写入以下内容：

  ```bash
  #!/bin/bash
  
  rm -f ../log/disk.tmp
  LANG=en_US
  # awk可以指定多个分隔符，这里指定空格和%为分隔符，从而去掉df -h中的%
  for r in `df -h | awk -F '[ %]+' '{print $5}' | grep -v Use`; do
      if [ $r -gt 90 ] && [ $send -eq "1" ]; then
          echo "$addr `date +%T` disk useage is $r" >> ../log/disk.tmp
      fi
  
      if [ -f ../log/disk.tmp ]; then
          df -h >> ../log/disk.tmp
          /bin/bash ../mail/mail.sh $addr\_disk $r ../log/disk.tmp
          echo "`date +%T` disk useage is no ok"
      else
          echo "`date +%T` disk useage is ok"
      fi
  done
  
  ```

---



