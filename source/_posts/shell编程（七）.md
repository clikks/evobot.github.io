---
title: shell编程（七）
author: Evobot
categories: Shell
tags:
  - Shell
abbrlink: cd013aaa
date: 2018-07-19 22:53:04
image:
---



expect是类似于shell的一种脚本语言，使用expect，我们可以实现自动登录远程机器并执行命令，下面介绍了如何使用expect远程登录服务器，以及在远程服务器上自动执行命令，并介绍了expect的参数用法。

<!--more-->

---

# expect介绍

## 分发系统介绍

- 当服务器数量较多时，上线业务如果逐台登录操作就会非常麻烦，所以就需要实现分发系统来批量操作；
- 这里使用`expect`来实现分发系统，expect是一种脚本语言，类似shell，用expect可以实现上传文件、远程执行命令等功能；
- 例如需要上线新的代码，使用expect实现，则需要准备一台模板机器，模板机器上的代码是最新的准备上线的代码，然后将需要上线的机器的ip和用户名密码记下来，使用expect脚本，借助rsync，将代码推送到需要上线的机器上去。

##  expect脚本远程登录

- 首先安装`expect`软件包，然后在`/usr/local/sbin`下创建``login.expect`脚本文件，写入以下内容：

  ```bash
  #!/usr/bin/expect

  # 创建变量host、passwd
  set host "192.168.67.130"
  set passwd "123456"

  # 执行系统shell命令
  spawn ssh root@$host

  # 截取系统输出，并执行针对的操作，ecp_continue表示继续
  expect {
  "yes/no" { send "yes\r"; exp_continue }
  "password:" { send "$passwd\r" }
  }
  # 表示脚本结束，停留在远程机器上
  interact

  ```

- 为脚本增加执行权限后，运行脚本，如果出现连接时等待时间太长，导致expect不能自动执行操作的问题，可以在脚本里设置超时时间，增加`set timeout 30`配置，即设置30秒内如果没有匹配到系统输出，就自动停止。

- `interact`表示执行完脚本后停留在远程机器上，如果不加这句，脚本执行完成后会立即退出远程机器，另外还有`expect eof`命令，会在执行完脚本后停留一两秒钟后再退出远程机器。

- 如果ssh登录时，需要等待很长时间才出现输入密码的提示，可以在目标服务器上，修改`sshd_config`，将`UseDNS` 设置为`no`即可。

## expect远程执行命令

- 在自动登录的基础上，可以在远程机器上执行命令，执行完成后退出远程，创建`command.expect`文件，写入以下内容：

  ```bash
  #!/usr/bin/expect

  set user "root"
  set passwd "123456"
  set host "192.168.67.130"
  set timeout 30

  spawn ssh $user@$host
  expect {
  "yes/no" { send "yes\r"; exp_continue }
  "password:" { send $passwd\r" }
  }

  # 匹配shell提示符
  expect "]*"
  send "touch /tmp/12.txt\r"
  expect "]*"
  send "echo 1212 > /tmp/12.txt\r"
  expect "]*"
  send "exit\r"

  ```

- 执行脚本结果如下：

  ```bash
  [root@load-balancer sbin]# ./command.expect 
  spawn ssh root@192.168.67.130
  root@192.168.67.130's password: 
  Last login: Thu Jul 19 23:57:17 2018 from 192.168.67.127
  [root@rs2 ~]# touch /tmp/12.txt
  [root@rs2 ~]# echo 1212 > /tmp/12.txt
  [root@rs2 ~]# [root@load-balancer sbin]# 

  ```

  在rs2机器上查看12.txt文件如下：

  ```bash
  [root@rs2 ~]# cat /tmp/12.txt 
  1212
  ```

  文件内容与我们脚本内定义的相同，说明命令都执行成功。

## expect脚本传递参数

- expect也可以传递参数，类似shell的`$1`、`$2`等，在expect脚本中，使用参数的方法如下面的脚本：

  ```bash
  #!/usr/bin/expect

  set user [lindex $argv 0]
  set host [lindex $argv 1]
  set passwd "123456"
  set cmd [lindex $argv 2]

  spawn ssh $user@$host 
  expect {
  "yes/no" { send "yes\r" }
  "password:" { send “$passwd\r" }
  }

  expect "]*"
  send "$cmd\r"
  expect "]*"
  send "exit\r"

  ```

  - 脚本中，参数从0开始，`[lindex $argv 0]`表示将第一个参数赋值给user，`[lindex $argv 1]`表示将第二个参数赋值给host，以此类推。

- 执行上面的脚本，结果如下：

  ```bash
  [root@load-balancer sbin]# ./para.expect root 192.168.67.130 "pwd;ls;vmstat 1 1"
  spawn ssh root@192.168.67.130
  root@192.168.67.130's password: 
  Last login: Fri Jul 20 00:41:50 2018 from 192.168.67.127
  [root@rs2 ~]# pwd;ls;vmstat 1 1
  /root
  123.sql  anaconda-ks.cfg  logs  master.zip  test_db-master  test_db-master.zip
  procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
   r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
   2  0      0 258196    876 145712    0    0    21     3   30   60  0  0 99  0  0
  [root@load-balancer sbin]# 

  ```

  - 在执行脚本的时候，后面跟对应的参数，而执行多条命令，可以使用将多条命令以`;`分割用`""`括起来，也可以在脚本中定义多个参数，每个参数对应一条命令。

---

