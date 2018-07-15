---
title: shell编程（三）
author: Evobot
categories: shell编程
tags:
  - shell编程
abbrlink: a2bcdace
date: 2018-07-14 00:09:24
image:
---



本文主要介绍shell脚本中，如何使用for循环和while循环，并且演示了continue和break的用法和区别，最后则介绍了exit的用法。

<!--more-->

---

# for循环

- shell编程中的for循环，语法为`for 变量名 in 条件; do ...; done`；

- 例如：

  ```bash
  #!/bin/bash
           
  sum=0   
  #seq产生两个数之间的所有整数
  for i in `seq 1 100`;do
  #shell中执行算数运算，$[]内前后不需要空格
      sum=$[$sum+$i]
  done     
  echo $sum              
  ```

- 在实际使用中，常用的还需要对文件列表进行循环，例如列出目录下所有的目录及其子目录：

  ```bash
  #!/bin/bash
       
  cd /etc/
  for a in `ls /etc/`;do
      #[ -d $a ] && ls $a
      if [ -d $a ];then
          echo "dir $a"
          ls $a
      fi
  done 

  ```

- for循环在进行文件遍历时，如果文件名中包含空格，将会存在下面这种情况：

  ```bash
  $ ll
  总用量 0
  -rw-rw-r-- 1 lux lux 0 7月  14 00:31  1.txt
  -rw-rw-r-- 1 lux lux 0 7月  14 00:31  2.txt
  -rw-rw-r-- 1 lux lux 0 7月  14 00:31 '3 4.txt'

  $ for i in `ls ./`;do echo $i;done
  1.txt
  2.txt
  3
  4.txt

  ```

  - `3 4.txt`文件名包含空格，在for循环中却被拆分成了两个文件，所以for循环会以空格和回车作为分隔符，需要在使用时特别注意。

# while循环

- shell中的while循环语法为`while 条件; do ...; done`；

- 示例1，监控系统负载并发邮件的脚本：

  ```bash
  #!/bin/bash 

  # 使用while true或while :创建死循环脚本
  while true;do                                                         # 获取系统1分钟的负载值  
      load=`w|head -1|awk -F 'load average: ' '{print $2}'|cut -d. -f1`
      if [ $load -gt 10 ];then
      # 调用发邮件脚本向邮箱发送邮件
          /usr/local/sbin/mail.py username@163.com 'load high' "$load"
      fi      
      sleep 30
  done     
  ```

- 示例2，检测用户输入的内容是否为数字：

  ```bash
  #!/bin/bash                     
        
  while true;do                   
      read -p "请输入一个数字:" n 
        
      if [ -z "$n" ];then           
          echo "未检测到输入内容" 
          continue                
      fi
        
      n1=`echo $n|sed 's/[0-9]//g'`
   
      if [ ! -z "$n1" ];then
          echo "只能输入纯数字"   
          continue                
      fi
      break                       
  done  
  echo $n                  
  ```

  - 其中`continue`表示跳过此次循环剩余部分，直接执行下一次循环；
  - `break`则表示跳出整个循环。

- 上面的脚本执行结果：

  ```bash
  $ bash test.sh
  请输入一个数字:4d
  只能输入纯数字
  请输入一个数字:
  未检测到输入内容
  请输入一个数字:23
  23

  ```

# break和continue的用法

## break

- break同样也可以用在for循环当中，例如：

  ```bash
  #!/bin/bash
           
  for i in `seq 1 5`;do
      echo $i
      # 如果if中的条件是判断是否是需要的字符串，则不能使用-eq，而应该使用[ $var == string ]
      if [ $i -eq 3 ];then
      	echo $i
          break
      fi
  done  
  echo 'finished'

  ```

  执行结果：

  ```bash
  $ bash break.sh 
  1
  2
  3
  3
  finished

  ```

## continue

- continue用于结束本次循环，执行下一次循环，同样也可以用在for循环中：

  ```bash
  #!/bin/bash
             
  for i in `seq 1 5`;do
      echo $i
      if [ $i -eq 3 ];then
          continue
      fi        
      echo $i                                                                                        
  done          
  echo 'finished'

  ```

  执行结果：

  ```bash
  $ bash break.sh
  1
  1
  2
  2
  3	# 由于continue的作用，后面的echo代码没有被执行
  4
  4
  5
  5
  finished

  ```

# exit的用法

- `exit`用来直接退出脚本，结束脚本的运行；

- 示例：

  ```bash
  #!/bin/bash

  for i in `seq 1 5`;do
      echo $i
      if [ $i -eq 3 ];then
          exit
      fi
      echo $i
  done
  echo "finished"

  ```

  执行结果：

  ```bash
  $ bash exit.sh 
  1
  1
  2
  2
  3
  # 指定到$i等于3时，脚本停止运行
  ```

- 通常使用时，`exit`会返回一个数字，如`exit 0`表示正常结束，`exit 1`等其他数字则表示异常退出。

---

