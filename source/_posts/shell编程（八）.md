---
title: shell编程（八）
author: Evobot
categories: shell编程
tags:
  - shell编程
abbrlink: 3a6a12e9
date: 2018-07-20 22:48:02
image:
---



1. expect脚本同步文件
2.  expect脚本指定host和要同步的文件
3. 构建文件分发系统
4. 批量远程执行命令

<!--more-->

---

# expect脚本同步文件

- 在expect中实现使用rsync自动同步文件，脚本的内容如下：

  ```bash
  #!/usr/bin/expect

  set passwd "123456"
  spawn rsync -av root@192.168.67.130:/tmp/12.txt /tmp
  expect {
  "yes/no" {send "yes\r"}
  "password:" { send "$passwd\r" }
  }
  expect eof
  ```

  执行结果：

  ```bash
  [root@load-balancer sbin]# ./sync.expect 
  spawn rsync -av root@192.168.67.130:/tmp/12.txt /tmp
  root@192.168.67.130's password: 
  receiving incremental file list
  12.txt

  sent 43 bytes  received 96 bytes  92.67 bytes/sec
  total size is 5  speedup is 0.04
  ```

  这里最后使用`expect eof`是为了防止文件还没传输完毕脚本就执行结束。

# expect脚本指定host及要同步的文件

- `对某个命令设置超时时间，只需要在send的命令之后增加一行`set timeout -1`表示永久不超时，或者`set timeout 3`表示三秒后超时；

- 指定host及需要同步的文件的脚本如下：

  ```bash
  #!/usr/bin/expect

  set passwd "123456"
  set host [lindex $argv 0]
  set file [lindex $argv 1]
  spawn rsync -av $file root@$host:$gile
  expect {
  "yes/no" { send "yes\r" }
  "password" { send "$passwd\r" }
  }
  expect eof

  ```

  执行结果：

  ```bash
  [root@load-balancer sbin]# ./host.expect 192.168.67.130 "/tmp/12.txt"
  spawn rsync -av /tmp/12.txt root@192.168.67.130:/tmp/12.txt
  root@192.168.67.130's password: 
  sending incremental file list

  sent 44 bytes  received 12 bytes  112.00 bytes/sec
  total size is 5  speedup is 0.09

  ```

  **需要注意的是，这种方式一次只能同步一个文件。**

---

# 构建文件分发系统

- 需求：

  对于线上业务来说，会经常更新网站代码或者配置文件，而且是大量服务器需要更新，少则几台，多则几十台，所以自动同步文件非常重要；

- 实现：

  需要一台模板机器，将要分发的文件准备好，然后使用expect脚本批量把要同步的文件分发到目标机器上即可。

- 核心命令：

  `rsync -av --files-from=list.txt  / root@host:/`

  `--files-from`指定保存有文件路径的文件，`list.txt`内保存需要同步的文件路径，文件路径必须使用绝对路径。

- 脚本如下：

  ```bash
  #!/usr/bin/expect

  set passwd "123456"
  set host [lindex $argv 0]
  set file [lindex $argv 1]

  # 源目录为根目录，目的目录也是根目录
  spawn rsync -av --files-from=$file / root@$host:/
  expect {
  "yes/no" { send "yes\r" }
  "password" { send "$passwd\r" }
  }
  expect eof

  ```

  **list.txt**的内容如下：

  ```bash
  /tmp/12.txt
  # 目录内的文件都会自动同步
  /usr/local/sbin/
  ```

  源目录的文件所在的目录，在目标机器上也必须有相同的路径和目录存在，如果目标机器没有相同的目录，可以在`rsync`中使用`rsync -avR`来自动创建不存在的目录。

- 存在多台服务器需要分发文件的时候，还需要创建一个`ip.txt`的文件来存储需要更新的服务器的IP地址，内容如下：

  ```bash
  192.168.67.130
  192.168.67.131
  127.0.0.1
  ```

- 然后创建`rsync.sh`文件，用来遍历ip地址并执行expect脚本：

  ```bash
  #!/bin/bash

  for ip in `cat /tmp/ip.txt`;do
      ./rsync.expect $ip /tmp/list.txt
  done

  ```

- 给`rsync.expect`增加执行权限，然后执行`rsync.sh`脚本，查看结果：

  ```bash
  $ bash -x rsync.sh 
  ++ cat /tmp/ip.txt
  + for ip in '`cat /tmp/ip.txt`'
  + ./rsync.expect 192.168.67.130 /tmp/list.txt
  spawn rsync -avR --files-from=/tmp/list.txt / root@192.168.67.130:/
  root@192.168.67.130's password: 
  building file list ... done
  tmp/
  usr/local/sbin/command.expect
  usr/local/sbin/host.expect
  usr/local/sbin/login.expect
  usr/local/sbin/lvs_dr.sh
  usr/local/sbin/lvs_nat.sh
  usr/local/sbin/para.expect
  usr/local/sbin/rsync.expect
  usr/local/sbin/rsync.sh
  usr/local/sbin/sync.expect

  sent 3,674 bytes  received 190 bytes  2,576.00 bytes/sec
  total size is 3,457  speedup is 0.89
  + for ip in '`cat /tmp/ip.txt`'
  + ./rsync.expect 127.0.0.1 /tmp/list.txt
  spawn rsync -avR --files-from=/tmp/list.txt / root@127.0.0.1:/
  root@127.0.0.1's password: 
  building file list ... done

  sent 400 bytes  received 12 bytes  824.00 bytes/sec
  total size is 3,457  speedup is 8.39

  ```

# 批量远程执行命令

- 在远程分发文件或者配置后，可能还需要执行一些重启服务之类的动作，所以还需要能够批量自动化的远程执行命令。

- expect实现功能脚本`exe.expect`内容如下,与单台机器执行命令相同：

  ```bash
  #!/usr/bin/expect

  set host [lindex $argv 0]
  set passwd "123456"
  set cmd [lindex $argv 1]

  spawn ssh root@$host
  expect {
  "yes/no" { send "yes\r" }
  "password" { send "$passwd\r" }
  }
  expect "]*"
  send "$cmd\r"
  expect "]*"
  send "exit\r"

  ```

- 接着创建`ext.sh`文件，用来循环IP并执行expect脚本：

  ```bash
  #!/bin/bash

  for ip in `cat /tmp/ip.list`; do
      ./exe.expect $ip "hostname"
  done
  ```

- 为脚本增加执行权限后，执行测试：

  ```bash
  [root@load-balancer sbin]# bash -x exe.sh 
  ++ cat /tmp/ip.txt
  + for ip in '`cat /tmp/ip.txt`'
  + ./exe.expect 192.168.67.130 hostname
  spawn ssh root@192.168.67.130
  root@192.168.67.130's password: 
  Last login: Sat Jul 21 01:29:59 2018 from 192.168.67.1
  [root@rs2 ~]# hostname
  rs2
  [root@rs2 ~]# + for ip in '`cat /tmp/ip.txt`'
  + ./exe.expect 127.0.0.1 hostname
  spawn ssh root@127.0.0.1
  root@127.0.0.1's password: 
  Last failed login: Sat Jul 21 01:27:33 CST 2018 from localhost on ssh:notty
  There were 2 failed login attempts since the last successful login.
  Last login: Sat Jul 21 00:41:13 2018 from 192.168.67.1
  [root@load-balancer ~]# hostname
  load-balancer

  ```

shell多线程：http://blog.lishiming.net/?p=448

---

