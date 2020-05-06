---
title: rsync通过服务同步及系统日志介绍、screen工具的使用
author: Evobot
categories: rsync
tags:
  - rsync
abbrlink: ccfd4295
date: 2018-05-15 22:03:12
image:
---



本文继续介绍rsync如何通过服务进行同步，另外介绍了Linux的系统日志和Screen工具的配置和使用。

<!--more-->

---

# rsync服务同步

- rsync通过服务同步的架构为CS架构，默认端口为873，启动服务后，机器之间可以通过服务端口进行通信和同步；

- 服务同步的格式为`rsync -av ... SRC host::[Module]`，如果配置文件中配置了`auth users`则命令为`rsync -av ...SRC [user]@host::[module]`，对应的配置文件为`/etc/rsyncd.conf`，配置文件内容如下：

  ```bash
  port=873	#指定rsync服务的监听端口
  log file=/var/log/rsync.log	
  pid file=/var/run/rsync.pid	
  address=192.168.199.141	#指定监听的本机ip，注释此项表示监听全部本机地址
  [test]	#模块名
  path=/tmp/rsync	#指定数据存放路径
  use chroot=true	
  #开启此选项后，连接时会chroot到path中，如果path中存在软连接文件，同步时又使用了-L选项，那么将会报错，这时可以将其设置为false
  max connections=4	#指定最大连接数，默认为0
  read only=true	#true表示path可写，false表示只读
  list=true	#表示当用户使用rsync -avP 192.168.199.141::查询该服务器的可用模块时，该模块是否被列出，建议设置为false
  uid=root
  gid=root #指定传输时以什么用户身份传输，即文件的uid/gid
  auth users=test	#指定传输时使用的用户名
  secrets file=/etc/rsyncd.passwd #指定auth users的密码文件，格式为username:passwd，权限为600
  hosts allow=192.168.199.142	#允许哪些服务器连接本机，多个IP使用空格分割，或者使用ip段：192.168.133.0/24
  ```

  > 指定的path必须要存在，命令中的module模块名指的是配置文件中的[test]配置。
  >
  > 如果指定了`auth users`和`secrets file`，对端机器在连接时可以创建rsync_pass.txt文件，并将密码写入其中，在同步时使用命令`rsync -av ...SRC --port=[port] --password-file=[/path/rsync_pass.txt][user]@host::[module]`，密码文件的权限为600。

- 启动rsync服务命令为`rsync --daemon`:

  ```bash
  [root@localhost ~]# rsync --daemon
  [root@localhost ~]# ps -aux | grep rsync
  root       9472  0.0  0.1 126292  1684 pts/0    S+   22:24   0:00 vi /etc/rsyncd.conf
  root       9502  0.0  0.0 114688   552 ?        Ss   22:34   0:00 rsync --daemon
  root       9506  0.0  0.0 112668   972 pts/1    S+   22:35   0:00 grep --color=auto rsync
  [root@localhost ~]# netstat -lntp
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1134/sshd           
  tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1709/master         
  tcp        0      0 192.168.199.141:873     0.0.0.0:*               LISTEN      9502/rsync          
  tcp6       0      0 :::22                   :::*                    LISTEN      1134/sshd           
  tcp6       0      0 ::1:25                  :::*                    LISTEN      1709/master
  ```

- 在另一台机器上执行命令进行同步：

  ```bash
  [root@localhost tmp]# rsync -avP /tmp/lab.swp 192.168.199.141::test/
  sending incremental file list
  lab.swp
      104,857,600 100%   47.92MB/s    0:00:02 (xfr#1, to-chk=0/1)

  sent 104,883,286 bytes  received 35 bytes  29,966,663.14 bytes/sec
  total size is 104,857,600  speedup is 1.00
  ```

  - 在配置文件内有`auth users`和`secrets file`两个配置，如果不想使用密码认证的话，可以注释这两行；
  - 如果在执行命令时报错提示如下：

  ```bash
  rsync: failed to connect to 192.168.199.140 (192.168.199.140): No route to host (113)
  rsync error: error in socket IO (code 10) at clientserver.c(125) [sender=3.1.2]
  ```

  - 那么可以安装`telnet`检查对端机器的rsync的端口是否开放，telnet命令格式为`telnet [ip address][port]`：

  ```bash
  [root@localhost tmp]# telnet 192.168.199.141 873
  Trying 192.168.199.141...
  telnet: connect to address 192.168.199.141: No route to host
  ```

  - 上面的输出表示对端端口无响应，然后我们需要检查对端机器的防火墙，可以关闭firewalld防火墙再重新telnet：

  ```bash
  [root@localhost tmp]# telnet 192.168.199.141 873
  Trying 192.168.199.141...
  Connected to 192.168.199.141.
  Escape character is '^]'.
  @RSYNCD: 31.0
  ^]
  telnet> quit
  Connection closed.
  ```

  - 上面的输出表示对端机器端口正常，这时候再重新执行`rsync`命令即可。

- 使用命令`rsync -avP host::[module]/files DEST`的形式从远端机器上拉取文件；

- 如果对端机器的rsync更改了端口，那么再同步时需要使用`--port=[port]`指定端口进行同步。

---

# Linux系统日志

## messages日志及logrotate介绍

- 系统日志会记录与系统有关的所有信息，能够帮助我们排查错误和了解机器状态；

- `/var/log/messages`日志是Linux系统的总日志，没有单独定义日志的服务，都会将日志记录在内，当messages日志达到一定大小时，系统会使用`logrotate`服务对其进行切割，logrotate的配置文件为`/etc/logrotate.conf`：

  ```bash
  [lux@evobot log]$ cat /etc/logrotate.conf
  # see "man logrotate" for details
  # rotate log files weekly
  weekly	# 表示每周切割一次

  # keep 4 weeks worth of backlogs
  rotate 4	#表示保留4个日志

  # create new (empty) log files after rotating old ones
  create	#切割完成时创建为新文件

  # use date as a suffix of the rotated file
  dateext	#新文件的后缀名，这里表示日期

  # uncomment this if you want your log files compressed
  #compress	#是否压缩

  # RPM packages drop log rotation information into this directory
  include /etc/logrotate.d	#包含指定目录下的其他配置文件

  # no packages own wtmp and btmp -- we'll rotate them here
  # 对指定日志进行切割的配置
  /var/log/wtmp {	
      monthly
      create 0664 root utmp	#创建时指定权限属主属组
          minsize 1M	#最小大小
      rotate 1	#保留个数
  }

  /var/log/btmp {
      missingok
      monthly
      create 0600 root utmp
      rotate 1
  }

  # system-specific logs may be also be configured here.
  ```

- 而在`/etc/logrotate.d`目录下，还有logrotate的其他配置文件：

  ```bash
  [lux@evobot log]$ ls /etc/logrotate.d/
  bootlog  chrony  nginx  syslog  wpa_supplicant  yum

  [lux@evobot log]$ cat /etc/logrotate.d/syslog
  /var/log/cron
  /var/log/maillog
  /var/log/messages
  /var/log/secure
  /var/log/spooler
  {
          sharedscripts
          dateext
          rotate 25
          size 40M
          compress
          dateformat  -%Y%m%d%s
          postrotate
                  /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true	#由于新建了日志文件，需要重启服务让日志写入到新的文件中
          endscript
  }
  ```

  > 关于logrotate的详细介绍，可以参考文章[logrotate 使用详解](https://my.oschina.net/u/2000675/blog/908189)

## logrotate示例

### 日志配置
- 通常不需要修改**logrotate.conf**文件，只需要将需要轮询的日志独立配置文件放在`/etc/logrotate.d/`目录下即可；
- 创建一个10MB的随机比特流数据日志文件：
  
  ```bash
  touch /var/log/test-log
  head -c 10M /dev/urandom > /var/log/test-log
  ```

- 在`/etc/logrotate.d/`目录下为日志文件创建同名配置文件：

  ```bash
  vi /etc/logrotate.d/test-log
  ```
  写入如下配置：
  ```bash
  /var/log/test-log {
      monthly
      rotate 5
      compress
      delaycompress
      missingok
      notifempty
      create 644 root root
      postrotate
          /usr/bin/killall -HUP rsyslogd
      endscript
  }
  ```
  
  - monthly: 日志文件将按月轮循。其它可用值为‘daily’，‘weekly’或者‘yearly’；
  - rotate 5: 一次将存储5个归档日志。对于第六个归档，时间最久的归档将被删除；
  - compress: 在轮循任务完成后，已轮循的归档将使用gzip进行压缩；
  - delaycompress: 总是与compress选项一起用，delaycompress选项指示logrotate不要将最近的归档压缩，压缩将在下一次轮循周期进行。这在你或任何软件仍然需要读取最新归档时很有用；
  - missingok: 在日志轮循期间，任何错误将被忽略，例如“文件无法找到”之类的错误；
  - otifempty: 如果日志文件为空，轮循不会进行；
  - create 644 root root: 以指定的权限创建全新的日志文件，同时logrotate也会重命名原始日志文件；
  - postrotate/endscript: 在所有其它指令完成后，postrotate和endscript里面指定的命令将被执行。在这种情况下，rsyslogd 进程将立即再次读取其配置并继续运行。
  
- 日志增长到指定大小进行分割：

  ```bash
  /var/log/test-log {
    size=50M
    rotate 5
    create 644 root root
    postrotate
        /usr/bin/killall -HUP rsyslogd
    endscript
}
  ```
  
- 分割的日志按照日期命名：
  
  ```bash
  /var/log/test-log {
    monthly
    rotate 5
    dateext
    create 644 root root
    postrotate
        /usr/bin/killall -HUP rsyslogd
    endscript
}
  ```
  
### logrotate操作

- logrotate可以使用命令手动调用，如为`/etc/logrotate.d/`目录下所有配置调用logrotate:

  ```bash
  # logrorate /etc/logrotate.conf
  ```
  
- 为某个特定配置调用：
  
  ```bash
  # logrotate /etc/logrotate.d/test-log
  ```


- logrotate命令具有`-d`选项，可以运行logrotate，但实际不会真正去操作日志，只会模拟打印结果：

  ```bash
  # logratatte -d /etc/logrotate.d/test-log
  ```
  
- 不满足日志轮询条件时，可以使用`-f`参数强制轮询,`-v`输出详细信息：
  
  ```bash
  # logrotate -vf /etc/logrotate.d/test-log
  ```

- logrotate自身的日志位于`/var/lib/logrotate/status/`目录，如果需要临时指定日志输出路径，使用如下命令：

  ```bash
  # logrotate -vf -s /var/log/logrotate-status /etc/logrotate.d/test-log
  ```
  
## dmsg日志

- `dmesg`命令可以查看系统的硬件相关的日志，dmesg日志是保存在内存中的，例如服务器有硬件故障，就需要查看这个日志，`dmesg -c`可以清空日志。
- 而`/var/log/dmesg`日志与`dmesg`命令查看的日志是不同的，其记录的是系统启动的相关日志。

## last/lastb命令

- `last`命令会调用`/var/log/wtmp`文件，这个文件为二进制文件，记录了系统所有的登陆日志，内容包括用户名、tty、来源ip，登陆时间和注销时间；
- 而`lastb`命令则调用`/var/log/btmp`文件，同样是二进制文件，记录了登陆失败的日志。

## secure日志

- `/var/log/secure`日志记录了登陆成功和登陆失败的详细日志，并会记录失败的具体原因。并且还记录了系统中其他一些安全相关的日志，如sudo命令、定时任务等等。

---

# screen工具

- Screen是个终端的多路复用器。可以在一个终端内执行任意多个应用程序。简单来说screen解决了终端下两个问题：

  1. 终端下工作有时会面临需要打开多个个终端，或者多个标签页的清空，每个里面运行一个应用或者完整某类功能。然后在终端之间需要经常切换切换，screen只需要一个终端就可以解决。
  2. 一般远程登录服务器，断网之后会使任务或命令中断，虽然可以使用`nohup command &`使命令在后台执行，但是这样会看不到命令的输出。而screen即使断网，或者放在后台、退出，都可以随时恢复。

- 使用screen之前需要安装`screen`软件包，直接执行`screen`命令就会创建一个虚拟终端，在其中可以任意执行任务，使用快捷键`ctrl+a+d`可以将screen虚拟终端放入后台，同时在虚拟终端中的任务不会终端，仍旧会继续执行；

- `screen -ls`可以查看所有的终端，Detached表示在后台运行，Attached表示在前台运行，每个终端最前面的数字为终端的ID：

  ```bash
  [root@evobot ~]# screen -ls
  There is a screen on:
          17491.pts-0.evobot      (Detached)
  1 Socket in /var/run/screen/S-root.
  ```

- 恢复Detached的screen终端，使用`screen -r [id]`：

  ```bash
  [root@evobot ~]# screen -r 17491
  ```

- 结束一个screen终端，可以在终端内使用`exit`命令结束，或者使用`ctrl+k`杀死当前终端；

- 创建screen终端还可以使用`screen -S [name]`来指定一个名字创建终端，这样在恢复终端时可以使用`screen -r [name]`来恢复终端。

---