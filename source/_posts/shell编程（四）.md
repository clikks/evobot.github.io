---
title: shell编程（四）
author: Evobot
categories: shell编程
tags:
  - shell编程
abbrlink: 1591c3ab
date: 2018-07-16 00:15:14
image:
---



本文主要介绍shell中的函数如何使用、并介绍了shell中的数组，以及数组的相关用法；最后则是准备shell脚本项目：监控系统的整体框架和组织架构。

<!--more-->

---

# shell中的函数

- 函数就是将一段代码整理成一个代码单元，并给这个单元起一个名字，即函数名，当使用这段函数时，只需要调用函数名即可，从而实现代码的复用；

- 函数的格式如下：

  ```bash
  # function关键字可以省略
  function func_name() {
      command
  }
  
  # 调用函数
  func_name
  ```

- 示例1，打印参数：

  ```bash
  #!/bin/bash
  
  function ipt() {
  	# $#表示参数个数，这里的$1等是指函数的参数，不是脚本的参数
      echo $1 $2 $3 $0 $#
  }
  
  ipt 1 a 2
  ```

  执行结果：

  ```bash
  $ bash func.sh
  1 a 2 func.sh 3
  ```

  - 如果想要在命令行中获得参数，可以将脚本修改为下面的形式：

  ```bash
  #!/bin/bash
  
  function ipt() {
      echo $1 $2 $3 $0 $#
  }
  
  # 将脚本的命令行参数传递给ipt函数
  ipt $1 $2 $3
  ```

  执行结果：

  ```bash
  $ bash func.sh abc cba bac dac
  abc cba bac func.sh 3
  # 由于脚本中只给ipt函数传递了三个参数，所以第四个参数不会被ipt所获取
  ```

- 示例2，加法计算：

  ```bash
  #!/bin/bash
  
  function sum() {
      s=$[$1+$2]
      echo $s
  }
  
  sum $1 $2
  ```

  执行结果

  ```bash
  $ bash func2.sh 5 6
  11
  ```

- 示例3，输入网卡名，获取IP地址：

  ```bash
  #!/bin/bash
  
  function get_ip() {
      ifconfig | grep -w -A1 "${1}:" | awk '/inet/ {print $2}'
  }
  
  read -p "Please input the eth name:" eth
  get_ip $eth
  
  ```

  执行结果：

  ```bash
  $ bash -x ip_get.sh
  + read -p 'Please input the eth name:' eth
  Please input the eth name:br0
  + get_ip br0
  + ifconfig
  + grep -w -A1 br0:
  + awk '/inet/ {print $2}'
  192.168.199.194
  
  $ bash ip_get.sh
  Please input the eth name:lo
  127.0.0.1
  ```

# shell中的数组

## 数组的元素

- 数组就是一串字符串或字符串的集合，并将其赋予变量，数组可以单独去除其中的一个元素，或者进行切片；

- 定义数组，语法为`a=(1 2 3 4 5 6)`或者`a=(1 'string' 3 4)`，打印数组，则是使用如下命令形式：
  
  ```bash
  echo ${a[@]}
  or 
  echo ${a[*]}
  ```

- 查看数组其中某个元素的值，则使用元素的下标，即元素在数组中是第几个，从0开始计数，例如：
  
  ```bash
  echo ${a[1]}
  or 
  echo ${a[0]}
  ```

- 如果要获得元素中所有元素的个数，则使用:

  ```bash
  echo ${#a[*]}
  ```

  ```bash
  $ a=(1 2 'str' 4 5)
  
  $ echo ${a[*]}
  1 2 str 4 5
  
  $ echo ${a[0]}
  1
  
  $ echo ${a[1]}
  2
  
  $ echo ${#a[*]}
  5
  
  ```

- 数组可以对其中的元素进行重新赋值，或者添加新的元素，重新赋值，语法为`a[1]=11`，添加元素，则是对不存在的下标进行赋值，如`a[5]=55`:

  ```bash
  $ a[1]=22
  
  $ echo ${a[*]}
  1 22 str 4 5
  
  $ a[5]=55
  
  $ echo ${a[*]}
  1 22 str 4 5 55
  
  ```

- 删除数组中的元素，则使用`unset`命令，如`unset a[3]`，清空整个数组，则是`unset a`：

  ```bash
  $ unset a[3]
  
  $ echo ${a[*]}
  1 22 str 5 55
  
  $ unset a
  
  $ echo ${a[*]}
  
  
  ```

## 数组的分片

- 数组分片就是从数组中截取某几个元素，如截取数组a中从下标0开始之后的3个元素，则为：

  ```bash
  echo ${a[*]:0:3}
  ```

  ```bash
  [root@vir1 ~]# b=(`seq 1 8`)
  
  [root@vir1 ~]# echo ${b[*]}
  1 2 3 4 5 6 7 8
  
  [root@vir1 ~]# echo ${b[*]:0:3}
  1 2 3
  ```

- 数组也可以从倒数的下标位进行切片，例如从倒数第三个元素开始，截取后面两个元素，则命令如下：

  ```bash
  # 倒数下标，必须写为0-n的形式表示
  [root@vir1 ~]# echo ${b[*]:0-4:3}
  5 6 7
  ```

- 对数组中的元素的值进行替换，其中对单个元素进行替换的效果与单独赋值是相同的，但当数组中存在相同值的元素时，替换能够将相同值的元素一次全部替换，数组替换的语法如下：

  ```bash
  echo ${b[*]/8/88}
  ```

  ```bash
  [root@vir1 ~]# b=(`seq 1 8`)
  
  [root@vir1 ~]# echo ${b[*]/8/88}
  1 2 3 4 5 6 7 88
  
  [root@vir1 ~]# b=(1 1 2 2 3 3)
  
  [root@vir1 ~]# echo ${b[*]}
  1 1 2 2 3 3
  
  [root@vir1 ~]# echo ${b[*]/1/11}
  11 11 2 2 3 3
  
  [root@vir1 ~]# echo ${b[*]/2/22}
  1 1 22 22 3 3
  
  ```

  - 通过上面最后两次的命名结果可以发现，对数组的元素的值进行替换并不会改变数组中原有的值，只是打印出替换后的值，如果想要原数组变成替换后的值，则使用下面的形式：

  ```bash
  # 用替换后的值重新创建数组并赋值为原数组变量
  [root@vir1 ~]# b=(${b[*]/1/11})
  
  [root@vir1 ~]# echo ${b[*]}
  11 11 2 2 3 3
  
  ```

# shell项目-告警系统需求分析

- 需求：使用shell定制各种个性化告警工具，但需要统一化管理、规范化管理；

- 思路：指定一个脚本包，包含主程序、子程序、配置文件、邮件引擎、输出日志等；

- 主程序：作为整个脚本的入口，是整个系统的命脉；

- 配置文件：是一个控制中心，用来开关各个子程序，指定各个相关联的日志文件；

- 子程序：监控脚本，用来监控各个指标；

- 邮件引擎：由python实现的邮件发送脚本，可以定义发件服务器、发件人和发件人密码；

- 输出日志：监控系统自己的运行日志输出。

- 要求：所有机器部署同样的监控系统，定制不通的配置文件；

- 程序架构如下：

  ![shell-mon](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/shell-mon.png)

  - bin目录下是主程序；conf目录下是配置文件；shares目录下是各个监控脚本；mail目录下是邮件引擎；log目录下则是日志。

---

