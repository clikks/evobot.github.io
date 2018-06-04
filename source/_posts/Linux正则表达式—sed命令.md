---
title: Linux正则表达式——sed命令
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
image:
abbrlink: 8d187ebb
date: 2018-04-26 21:13:33
---



**sed**是stream editor的简称上是一个编辑器，但是它是非交互式的流编辑器。

sed的处理流程如下：

![sed处理流程](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/sed.png)

1. 读入一行内容到模式空间(Pattern Space)
2. 对模式空间里的内容执行sed命令
3. 输出模式空间的内容，当输出内容后，模式空间将被清空
4. 重复以上的操作，直到文件的最后一行。

<!--more-->

---

# **sed**命令的用法

## 匹配用法

- `sed`命令可以用来对文件内容进行查找替换，最简单的使用命令方法为`sed '/pattern/'p [filename]`，其中`p`表示打印的意思，使用`-n`参数可以不显示多余的内容：

  ```bash
  [root@evobot ~]# sed -n '/root/'p passwd 
  root:x:0:0:root:/root:/bin/bash
  ```

- `sed`命令同样也支持正则表达式，但是对扩展正则元字符需要使用脱义符脱义，或者使用`-r`参数：

  ```bash
  [root@evobot ~]# sed -n '/o\+t/'p passwd 
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/rooooot:/sbin/nologin
  evobot:x:1002:1002::/home/evobot:/bin/bash

  [root@evobot ~]# sed -nr '/o+t/'p passwd 
  root:x:0:0:root:/root:/bin/bash
  operator:x:11:0:operator:/rooooot:/sbin/nologin
  evobot:x:1002:1002::/home/evobot:/bin/bash
  ```

- `sed`除了进行匹配外，还可以打印指定的行，或使用`,`分割指定行号范围，使用`$`表示末行：

  ```bash
  [root@evobot ~]# sed -n '2'p passwd 
  bin:x:1:1:bin:/bin:/sbin/nologin

  [root@evobot ~]# sed -n '2,5'p passwd 
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin

  [root@evobot ~]# sed -n '25,$'p passwd 
  syslog:x:996:994::/home/syslog:/bin/false
  centos:x:1000:1000:Cloud User:/home/centos:/bin/BASH
  lux:x:1001:1001::/home/lux:/bin/bash
  nginx:x:995:993:Nginx web server:/var/lib/nginx:/sbin/nologin
  evobot:x:1002:1002::/home/evobot:/bin/bash
  ```

- `sed -e`参数表示在一个表达式内执行多个操作，如打印指定行并匹配指定字符串，多个动作时间使用`-e`参数，如果多个表达式匹配内容相同，会打印多次：

  ```bash
  # 打印第一行并匹配'evobot'字符串所在的行
  [root@evobot ~]# sed -e '1'p -e '/evobot/'p -n passwd 
  root:x:0:0:root:/root:/bin/bash
  evobot:x:1002:1002::/home/evobot:/bin/bash
  ```

- `sed`可以在匹配时不区分大小写，命令格式为`sed -n '/pattern/Ip [filename]'`：

  ```bash
  [root@evobot ~]# sed -n '/bus/'Ip passwd 
  sync:x:5:0:BUS:/sbin:/bin/sync
  dbus:x:81:81:System message bus:/:/sbin/nologi
  ```

## 删除用法

- `sed 'n,m'd [file]`可以将指定的行`n`或行数范围`n~m`行删除，例如对于一个日志文件，只需要指定日期之后的内容，可以先使用`grep -n`查看指定日期的行号，再删除之前的行：

  ```bash
  [root@evobot ~]# grep -n 'Apr 24' secure.log 
  180652:Apr 24 00:23:31 evobot sshd[9431]: Received disconnect from 118.113.33.203 port 11905:11: disconnected by user
  180653:Apr 24 00:23:31 evobot sshd[9431]: Disconnected from 118.113.33.203 port 11905
  180654:Apr 24 00:23:31 evobot sshd[9429]: pam_unix(sshd:session): session closed for user lux
  180655:Apr 24 00:23:31 evobot sshd[2268]: Received disconnect from 118.113.33.203 port 11968:11: disconnected by user
  180656:Apr 24 00:23:31 evobot sshd[2268]: Disconnected from 118.113.33.203 port 11968
  180657:Apr 24 00:23:31 evobot sshd[2266]: pam_unix(sshd:session): session closed for user lux
  180658:Apr 24 00:23:31 evobot su: pam_unix(su-l:session): session closed for user root
  180659:Apr 24 04:08:30 evobot sshd[22139]: Did not receive identification string from 183.57.54.43 port 55728
  180660:Apr 24 22:24:39 evobot sshd[3396]: Accepted publickey for lux from 118.113.33.203 port 8769 ssh2: RSA SHA256:Y7Fo5sjyHH9pIUnCz+uLdIIT/2wmR4lPn3Wh5x8H9O4
  180661:Apr 24 22:24:39 evobot sshd[3396]: pam_unix(sshd:session): session opened for user lux by (uid=0)
  180662:Apr 24 22:25:03 evobot sudo:     lux : TTY=pts/0 ; PWD=/home/lux ; USER=root ; COMMAND=/bin/su -
  180663:Apr 24 22:25:03 evobot su: pam_unix(su-l:session): session opened for user root by lux(uid=0)

  # 使用sed删除前180651行
  [root@evobot ~]# sed '1,180651'd secure.log 
  Apr 24 00:23:31 evobot sshd[9431]: Received disconnect from 118.113.33.203 port 11905:11: disconnected by user
  Apr 24 00:23:31 evobot sshd[9431]: Disconnected from 118.113.33.203 port 11905
  Apr 24 00:23:31 evobot sshd[9429]: pam_unix(sshd:session): session closed for user lux
  Apr 24 00:23:31 evobot sshd[2268]: Received disconnect from 118.113.33.203 port 11968:11: disconnected by user
  Apr 24 00:23:31 evobot sshd[2268]: Disconnected from 118.113.33.203 port 11968
  Apr 24 00:23:31 evobot sshd[2266]: pam_unix(sshd:session): session closed for user lux
  Apr 24 00:23:31 evobot su: pam_unix(su-l:session): session closed for user root
  Apr 24 04:08:30 evobot sshd[22139]: Did not receive identification string from 183.57.54.43 port 55728
  Apr 24 22:24:39 evobot sshd[3396]: Accepted publickey for lux from 118.113.33.203 port 8769 ssh2: RSA SHA256:Y7Fo5sjyHH9pIUnCz+uLdIIT/2wmR4lPn3Wh5x8H9O4
  Apr 24 22:24:39 evobot sshd[3396]: pam_unix(sshd:session): session opened for user lux by (uid=0)
  Apr 24 22:25:03 evobot sudo:     lux : TTY=pts/0 ; PWD=/home/lux ; USER=root ; COMMAND=/bin/su -
  Apr 24 22:25:03 evobot su: pam_unix(su-l:session): session opened for user root by lux(uid=0)
  ```

- 上面的删除方式只是将删除结果显示在屏幕上，并没有真正在文件内进行删除操作，使用`-i`选项可以直接更改文件内容，命令为`sed -i 'n,m'd [file]`：

  ```bash
  [root@evobot ~]# wc -l secure.log
  180663 secure.log
  [root@evobot ~]# sed -i '1,180651'd secure.log
  [root@evobot ~]# wc -l secure.log
  12 secure.log
  ```

## 替换用法

- `sed`命令的替换用法与vim中的替换相似，命令用法为`sed s/book/books/g [file]`，其中`s`表示替换，`g`表示全局替换，不加`g`表示只替换每行匹配到的第一个关键字；

- 在替换时，也可以指定范围进行替换，命令为`sed 'n,ms/book/books/g' [file]` :

  ```bash
  [root@evobot ~]# cat secure.log
  Apr 24 00:23:31 evobot sshd[9431]: Received disconnect from 118.113.33.203 port 11905:11: disconnected by user
  Apr 24 00:23:31 evobot sshd[9431]: Disconnected from 118.113.33.203 port 11905
  Apr 24 00:23:31 evobot sshd[9429]: pam_unix(sshd:session): session closed for user lux

  [root@evobot ~]# sed 's/evobot/sedtest/g' secure.log
  Apr 24 00:23:31 sedtest sshd[9431]: Received disconnect from 118.113.33.203 port 11905:11: disconnected by user
  Apr 24 00:23:31 sedtest sshd[9431]: Disconnected from 118.113.33.203 port 11905
  Apr 24 00:23:31 sedtest sshd[9429]: pam_unix(sshd:session): session closed for user lux

  [root@evobot ~]# sed '8,$s/evobot/mylinux/g' secure.log
  Apr 24 00:23:31 evobot su: pam_unix(su-l:session): session closed for user root
  Apr 24 04:08:30 mylinux sshd[22139]: Did not receive identification string from 183.57.54.43 port 55728
  Apr 24 22:24:39 mylinux sshd[3396]: Accepted publickey for lux from 118.113.33.203 port 8769 ssh2: RSA SHA256:Y7Fo5sjyHH9pIUnCz+uLdIIT/2wmR4lPn3Wh5x8H9O4
  Apr 24 22:24:39 mylinux sshd[3396]: pam_unix(sshd:session): session opened for user lux by (uid=0)
  Apr 24 22:25:03 mylinux sudo:     lux : TTY=pts/0 ; PWD=/home/lux ; USER=root ; COMMAND=/bin/su -
  Apr 24 22:25:03 mylinux su: pam_unix(su-l:session): session opened for user root by lux(uid=0)
  ```

- 同样的在替换时也可以使用正则表达式进行匹配替换：

  ```bash
  [root@evobot ~]# head -n 5 secure.log
  Apr 24 00:23:31 evobot sshd[9431]: Received disconnect from 118.113.33.203 port 11905:11: disconnected by user
  Apr 24 00:23:31 evobot sshd[9431]: Disconnected from 118.113.33.203 port 11905
  Apr 24 00:23:31 evobot sshd[9429]: pam_unix(sshd:session): session closed for user lux
  Apr 24 00:23:31 evobot sshd[2268]: Received disconnect from 118.113.33.203 port 11968:11: disconnected by user
  Apr 24 00:23:31 evobot sshd[2268]: Disconnected from 118.113.33.203 port 11968

  [root@evobot ~]# sed -r '1,5s/[0-9]+.[0-9]+.[0-9]+.[0-9]+/ipaddress/g' secure.log
  Apr ipaddress evobot sshd[9431]: Received disconnect from ipaddress port ipaddress: disconnected by user
  Apr ipaddress evobot sshd[9431]: Disconnected from ipaddress port 11905
  Apr ipaddress evobot sshd[9429]: pam_unix(sshd:session): session closed for user lux
  Apr ipaddress evobot sshd[2268]: Received disconnect from ipaddress port ipaddress: disconnected by user
  Apr ipaddress evobot sshd[2268]: Disconnected from ipaddress port 11968
  ```

- `sed`也支持与管道符连用，对管道符前面的命令输出内容进行处理；

- `sed`可以使用`()`包裹关键词进行匹配，`()`匹配到的内容，可以使用`\1`这样的形式调用，比如将开头一段和结尾一段字符串进行调换：

  ```bash
  [root@evobot ~]# head -n 3 passwd 
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin

  [root@evobot ~]# head -n 3 passwd | sed -r '1,3s/(\w+):(.*):([^:]+)/\3:\2:\1/g'	# \w表示字符或数字
  /bin/bash:x:0:0:root:/root:root
  /sbin/nologin:x:1:1:bin:/bin:bin
  /sbin/nologin:x:2:2:daemon:/sbin:daemon
  ```

- `sed`中如果匹配的内容中包括`/`，则需要使用转义符`\`转义，或者将`sed`的分隔符换成`@`、`:`或`|`:

  ```bash
  [root@evobot ~]# head -n 3 passwd 
  root:x:0:0:root:/root:/bin/BASH
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  [root@evobot ~]# head -n 3 passwd | sed -r 's|/[a-z]+/[a-z]+|/home/bin|Ig'
  root:x:0:0:root:/root:/home/bin
  bin:x:1:1:bin:/bin:/home/bin
  daemon:x:2:2:daemon:/sbin:/home/bin
  ```

- 替换时也可以不指定替换结果的字符串，则替换成空：

  ```bash
  [root@evobot ~]# head -n 3 passwd | sed 's/[a-zA-Z]//g'
  ::0:0::/://
  ::1:1::/://
  ::2:2::/://
  ```

- 为所有行开头添加指定字符串，可以使用`sed -r 's/(.*)/xxx:&/'`表示，其中`&`的含义与前面使用的`\1`含义相同：

  ```bash
  [root@evobot ~]# head -n 3 passwd | sed -r 's/^/test:&/'
  test:root:x:0:0:root:/root:/bin/BASH
  test:bin:x:1:1:bin:/bin:/sbin/nologin
  test:daemon:x:2:2:daemon:/sbin:/sbin/nologin
  ```

- 同样也可以在结尾增加内容：

  ```bash
  [root@evobot ~]# head -n 3 passwd | sed -r 's/$/&:test/'
  root:x:0:0:root:/root:/bin/BASH:test
  bin:x:1:1:bin:/bin:/sbin/nologin:test
  daemon:x:2:2:daemon:/sbin:/sbin/nologin:test
  ```

---