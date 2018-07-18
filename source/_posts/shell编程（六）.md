---
title: shell编程（六）
author: Evobot
date: 2018-07-18 23:40:35
categories: shell编程
tags:
  - shell编程
image:
---



本文将邮件引擎的进行了详细的介绍，以及如何用脚本进行告警收敛，使得告警邮件不会持续发送，另外针对运行告警监控系统进行了介绍。

<!--more-->

---

# 告警系统邮件引擎

- 在mail目录内创建`mail.py`文件，写入python发送邮件脚本内容：

  ```python
  #!/usr/bin/env python
  #-*- coding: UTF-8 -*-
  import os,sys
  reload(sys)
  sys.setdefaultencoding('utf8')
  import getopt
  import smtplib
  from email.MIMEText import MIMEText
  from email.MIMEMultipart import MIMEMultipart
  from  subprocess import *
  
  def sendqqmail(username,password,mailfrom,mailto,subject,content):
      gserver = 'smtp.163.com'
      gport = 465
  
      try:
          msg = MIMEText(unicode(content).encode('utf-8'))
          msg['from'] = mailfrom
          msg['to'] = mailto
          msg['Reply-To'] = mailfrom
          msg['Subject'] = subject
  
          smtp = smtplib.SMTP_SSL(gserver, gport)
          smtp.set_debuglevel(0)
          smtp.ehlo()
          smtp.login(username,password)
  
          smtp.sendmail(mailfrom, mailto, msg.as_string())
          smtp.close()
      except Exception,err:
          print "Send mail failed. Error: %s" % err
  
  
  def main():
      to=sys.argv[1]
      subject=sys.argv[2]
      content=sys.argv[3]
  ##定义QQ邮箱的账号和密码，你需要修改成你自己的账号和密码（请不要把真实的用户名和密码放到网上公开，否则你会死的很惨)
      sendqqmail('username@163.com','password','username@163.com',to,subject,content)
  
  if __name__ == "__main__":
      main()
  
  
  #####脚本使用说明######
  #1. 首先定义好脚本中的邮箱账号和密码
  #2. 脚本执行命令为：python mail.py 目标邮箱 "邮件主题" "邮件内容"
  
  ```

- 在mail目录中再创建`mail.sh`脚本文件，写入内容如下：

  ```bash
  #!/bin/bash
  
  # $1就是前面的监控脚本中的$addr\_disk等参数
  log=$1
  t_s=`date +%s`
  # 获取两小时之前的时间戳是为了让发生告警时第一次执行此脚本能够满足大于3600的条件，从而发送邮件
  t_s2=`date -d "2 hours ago" +%s`
  
  # 如果日志文件不存在，则创建并写入两小时之前的时间戳
  if [ ! -f /tmp/$log ]; then
      echo $t_s2 > /tmp/$log
  fi
  
  # 截取日志最后一行的时间戳
  t_s2=`tail -1 /tmp/$log | awk '{print $1}'`
  echo $t_s >> /tmp/$log
  v=$[$t_s-$t_s2]
  echo $v
  
  # 如果上次告警距离现在超过1小时，则发送告警邮件，并将$log.txt置零
  if [ $v -gt 3600 ]; then
      ./mail.py $1 $2 $3
      echo "0" > /tmp/$log.txt
  else
      if [! -f /tmp/$log.txt ];then
          echo "0" > /tmp/$log.txt
      fi
      nu=`cat /tmp/$log.txt`
      # 每分钟给计数器文件加1
      nu2=$[$nu+1]
      ehco $n2 > /tmp/$log.txt
      # 当超过10分钟时，继续发送邮件提示告警，并将计数器置零
      if [ $nu2 -gt 10 ];then
          ./mail.py $1 "trouble continue 10 min $2" "$3"
          echo "0" > /tmp/$log.txt
      fi
  fi
  ```

  - 上面的脚本的执行逻辑是，当监控脚本第一次监控到告警，执行此脚本，首先获取当前的时间戳和两小时前的时间戳，由于是第一次执行，`tmp/$log`文件不存在，所以将两小时前的时间戳写入`$log`，然后从`$log`文件中获取时间戳重新赋值给`t_s2`变量，并将当前时间的时间戳`t_s`写入到`$log`，记录此次告警发生的时间戳，然后计算`t_s2`和`t_s`的差值，由于第一次运行，`t_s2`是两小时之间的时间戳，所以差值肯定大于3600，所以立即发送邮件，并创建`$log.txt`文件，写入计数器0；
  - 监控脚本每分钟执行一次，第二次执行时，再次获取当前时间戳和两小时之前的时间戳，这时`$log`文件存在，并且`$log`的内容是上次上次告警发生时的`t_s`的时间戳，获取这个时间戳并覆盖变量`t_s2`，重新计算差值，由于只过了1分钟，差值小于3600，则判断`$log.txt`是否存在，存在的话获取其中的值并加1，如果加1后小于10，则说明两次告警的发生时间没有超过10分钟，则不再发送邮件；
  - 当加1后大于10，说明告警已经持续了10分钟，所以再次发送邮件，并将`$log.txt`重新归零。
  - 实际上如果两次告警之间的时间不超过一个小时，而计数器又不超过10，那么也不会发送邮件给管理员，所以这种情况下，不是持续十分钟发送告警邮件，而是告警发生时间之间只要不超过一小时，都必须连续告警10次之后，才会发送邮件。

# 运行告警系统

- 为告警系统的主脚本设置定时任务，每分钟执行一次；

- 首先给`mail.py`增加执行权限，然后因为`mail.py`中，定义的三个参数分别为收件人、邮件主题、邮件内容，所以还需要修改监控脚本中发送邮件的部分，以`load.sh`为例，将发送邮件的部分修改如下：

  ```bash
  /bin/bash ../mail/mail.sh 15608013032@163.com "$addr\_load:$load" `cat ../log/load.tmp`
  ```

- 执行`crontab -e`，然后写入以下规则：

  ```bash
  * * * * * cd /usr/local/sbin/mon/bin; bash main.sh
  ```

- 执行结果可以在`log`目录下查看`err.log`确认执行中是否出错，结果可以在`mon.log`中查看。

---

