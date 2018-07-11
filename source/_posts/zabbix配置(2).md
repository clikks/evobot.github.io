---
title: zabbix配置(2)
author: Evobot
date: 2018-07-11 00:40:23
categories: zabbix
tags:
  - zabbix
  - 监控
image:

---

本文主要介绍如何在zabbix中添加自定义监控项目，以及怎么配置zabbix告警的邮件提醒。

<!--more-->

---

# 添加自定义监控项目

- 对于模板内没有的监控项目，就需要自定义监控，例如监控某台web服务器的80端口的连接数，并出图；

- 首先编写脚本`/usr/local/sbin/estab.sh`，内容如下：

  ```bash
  #! /bin/bash
  ##获取80端口并发连接数
  netstat -ant | grep ':80' |grep -c ESTABLISHED
  ```

- 然后修改脚本权限为755，因为这个脚本将由zabbix用户来执行；

- 接着在客户端上编辑｀/etc/zabbix/zabbix_agentd.conf`,查找｀UnsafeUserParameters｀配置项，该配置的值为１时表示使用自定义脚本：

  ```bash
  # 在配置文件中修改该配置为１
  UnsafeUserParameters=1
  ```

- 然后继续在配置文件中搜索`UserParameter`，该配置用来定义自定义脚本的路径和名字：

  ```bash
  UserParameter=my.estab.count[*],/usr/local/sbin/estab.sh
  ```

  > 其中my.estab.count是监控项中的键值，在新建监控项时要填入这个自定义的名字，[*]表示脚本执行没有参数，如果有参数，则使用逗号分割参数写在[]中。

- 然后重启zabbix-agent，再到服务端执行下面的命令对自定义的脚本进行测试：

  ```bash
  $ zabbix_get -s 192.168.67.130 -p 10050 -k 'my.estab.count'
  0

  ```

  > 测试结果应该能够得到正常的数字，否则存在问题。-k是指定键值。

- 接着到监控中心配置监控项目，点击主机>监控项>创建监控项：

  ![zabbix-mon](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-mon.png)

- 完成后点击图形，按照之前添加图形的方式，将新增的监控项创建一个图形。

- 然后创建触发器，点击触发器>创建触发器，严重性选择警告，如下图，点击添加即可：

  ![zabbix-alarm](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-alarm.png)

# 配置邮件告警

## 创建邮件脚本

- 使用163等邮箱发送告警邮件，首先登录个人邮箱，设置开启POP3、IMAP、SMTP服务，开启并记录授权码；

- 然后到**管理** > **报警媒介类型** > **创建媒体类型**，然后填入名称，类型选择为脚本，脚本名称为自定义的`mail.py`，如下图：

  ![zabbix-mail](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-mail.png)

  - 其中脚本参数中，第一个表示收件人，第二个是主题，第三个是邮件内容，这三个参数就是python脚本的参数。

- 创建报警脚本mail.py，而脚本存放位置，则是在服务端的配置文件中定义的，在配置文件的` AlertScriptsPath=/usr/lib/zabbix/alertscripts`配置，其定义的路径就是脚本的存放路径；

- 在脚本目录下创建mail.py文件，写入如下内容：

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
      gport = 25

      try:
          msg = MIMEText(unicode(content).encode('utf-8'))
          msg['from'] = mailfrom
          msg['to'] = mailto
          msg['Reply-To'] = mailfrom
          msg['Subject'] = subject

          smtp = smtplib.SMTP(gserver, gport)
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
  ##定义邮箱的账号和密码，你需要修改成你自己的账号和密码.
      sendqqmail('username@163.com','mypassword','username@163.com',to,subject,content)

  if __name__ == "__main__":
      main()

  #####脚本使用说明######
  #1. 首先定义好脚本中的邮箱账号和密码
  #2. 脚本执行命令为：python mail.py 目标邮箱 "邮件主题" "邮件内容"

  ```

- 脚本完成后，将权限设置为755，然后可以使用下面的命令测试能否收到邮件：

  ```bash
  python mail.py myusername@163.com "test mail" "test message"
  ```

## 创建用户

- 在管理》用户界面中，点击创建用户，用户的群组可以选择`Zabbix administrators`，然后设置密码，中文；

- 接着点击用户二级菜单的报警媒介》添加，选择我们创建的新的脚本报警媒介，将收件人邮箱填上，其余全部打钩，点击添加》更新：

  ![zabbix-mail2](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-mail2.png)

- 接着点击用户群组》Zabbix administrators 》权限，操作如下图：

  ![zabbix-mail3](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-mail3.png)

## 动作配置

- 点击配置》动作，是用来触发器触发之后，需要执行的事情，如发邮件；

- 点击创建动作，填入名称，而条件中的`维护状态 非在 维护`则是表示当主机被设置为维护状态时，监控项不触发告警，防止误报；

- 在新的触发条件中，添加一个`触发器示警度>=未分类`,这样当监控主机不在维护状态时，并且有告警，不论告警级别，都会发送邮件：

  ![zabbix-act](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-act.png)

- 接着点击**操作**，在默认信息中，修改内容如下：

  ```bash
  HOST:{HOST.NAME} {HOST.IP}
  TIME:{EVENT.DATA} {EVENT.TIME}
  LEVEL:{TRIGGER.SEVERITY}
  NAME:{TRIGGER.NAME}
  messages:{ITEM.NAME}:{ITEM.VALUE}
  ID:{EVENT.ID}
  ```

  > 其中，HOST项表示定义的hostname和ip，TIME表示告警时间，LEVEL则表示示警度，NAME则是触发器的key，messages则是告警的状态码，ID则是事件的ID。

- 接着在操作细节中的配置如下图，选择发送消息操作类型，并且发送到用户，选择我们创建的新用户，并且增加条件：

  ![zabbix-act2](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-act2.png)

- 然后点击**恢复操作**，将默认信息的内容改为如下内容：

  ```bash
  HOST:{HOST.NAME} {HOST.IP}
  TIME:{EVENT.DATA} {EVENT.TIME}
  LEVEL:{TRIGGER.SEVERITY}
  NAME:{TRIGGER.NAME}
  messages:{ITEM.NAME}:{ITEM.VALUE}
  ID:{EVENT.ID}
  ```

- 并且在操作细节中，配置与**操作**的配置相同：选择发送到用户，仅发送到自定义的脚本报警媒介；

- 完成添加后，新添加的动作状态为已启用。

# 测试告警

- 首先将触发器的触发条件降低，如80端口的并发连接数触发器的表达式设置为大于等于1；

- 然后在仪表板中等待一会，会在主机状态和最近问题中显示问题；

- 当发送了邮件时，会在动作列显示完成，此时到邮箱查看邮件即可。

  ![zabbix-test](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/zabbix-test.png)

- 另外在**配置**》**主机**》**监控项**中，打开需要配置的监控项，可以在类型中选择主被动模式，默认为zabbix客户端，即被动模式，而zabbix客户端(主动式)则是主动模式。