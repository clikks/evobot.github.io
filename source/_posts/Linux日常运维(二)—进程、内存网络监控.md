---
title: Linux日常运维(二)——进程、内存网络监控
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 9f02128c
date: 2018-05-06 22:49:49
image:
---



本文主要介绍LInux日常运维中如何监控io性能，以及free查看内存的命令和ps查看进程的命令，另外介绍如何查看网络状态并且对网络进行抓包的相关知识。

<!--more-->

---

# 磁盘监控

## **iostat**命令

- 使用`vmstat`查看系统状态时，如果bi、bo或者wa列数值过高，那么需要查看磁盘更详细的状态，就可以使用`iostat`命令；

- `iostat`命令是在安装`sysstat`软件包时一并安装到系统中的，直接执行`iostat`命令可以查看系统中各个磁盘的详细信息，也可以像`vmstat`命令一样执行`iostat 1`命令每秒刷新输出磁盘信息；

  ```bash
  [root@evobot ~]# iostat 
  Linux 3.10.0-693.21.1.el7.x86_64 (evobot) 	2018年05月06日 	_x86_64_	(1 CPU)

  avg-cpu:  %user   %nice %system %iowait  %steal   %idle
             0.61    0.00    0.24    0.29    0.00   98.85

  Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
  vda               2.04         0.29        18.04     454791   27963960
  scd0              0.00         0.00         0.00        552          0
  ```

- 列出的信息中，每列分别表示设备名、每秒进程下发的IO读写请求数量、每秒读取的kB数，每秒写入的kB数、取样时间间隔内读取的kB总数和取样时间间隔内写入的kB总量；

- 上面的输出信息与`sar -b`查看到的内容是一样的，更常用的用法为`iostat -x 1`:

  ```bash
  [root@evobot ~]# iostat -x
  Linux 3.10.0-693.21.1.el7.x86_64 (evobot) 	2018年05月06日 	_x86_64_	(1 CPU)

  avg-cpu:  %user   %nice %system %iowait  %steal   %idle
             0.61    0.00    0.24    0.29    0.00   98.85

  Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await r_await w_await  svctm  %util
  vda               0.00     2.36    0.01    2.03     0.29    18.04    17.98     0.03   16.11    9.70   16.16   1.77   0.36
  scd0              0.00     0.00    0.00    0.00     0.00     0.00     9.95     0.00    1.41    1.41    0.00   1.41   0.00
  ```

  - 这里列出的信息，主要关注`%util`列，其表示**进程被I/O需求消耗的CPU百分比**，如果这里的值超过50%，但wkB/s和rkB/s比较小，则说明磁盘io繁忙，磁盘存在一些问题，可能需要更换磁盘；

## **iotop**命令

- 当我们使用`iostat`查看到磁盘io较高时，可能需要查看是哪个进程的磁盘读写较高，就可以使用`iotop`命令，使用`iotop`命令需要安装`iotop`软件包；

- 执行`iotop`命令，其命令输出类似于`top`命令，每秒钟刷新一次：

  ![iotop](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/iotop.png)

- 主要关注的是`IO`列和`COMMAND`列，来查看具体哪个进程占用IO比例最高；

---

# 内存和进程监控

## **free**命令

- `free`命令主要用来查看内存的使用情况：

  ```bash
  [root@evobot ~]# free
                total        used        free      shared  buff/cache   available
  Mem:        1883540       97652      468720         392     1317168     1586592
  Swap:             0           0           0
  ```

- 所列出的信息中，第一行为标题，第二行为内存使用情况，第三行为交换分区使用情况；
- 各列所表示信息如下表，默认`free`命令输出信息单位为`kB`，可以使用`free -m`转换单位为MB，或者使用`free -h`人性化显示：

<style>
table th {
    text-align: center;
}
table th:first-of-type {
    width: 120px;
}
</style>

| 标示         | 说明                                       |
| :----------: | ---------------------------------------- |
| total      | 内存总大小                                    |
| used       | 内存已被使用的大小                                |
| free       | 空闲内存大小                                   |
| shared     | 共享的内存大小                                  |
| buff/cache | 缓冲(buff)及缓存(cache)大小，cache是从磁盘预先读取出来准备送往CPU处理的数据，buf则是CPU处理完的准备存储到磁盘上的数据 |
| available  | 表示空闲内存加上未分配给buff/cache的内存大小              |

- `free`命令的输出中，total=used+free+buff/cache，而avaliable=free+(buff/cache剩余部分)，所以在查看内存状态时，应该关注的是**avaliable**的值。
- 而swap行如果占用过高，或者swap剩余为0，则需要增加内存，或者有程序存在内存泄露，需要进一步排查。

## **ps**命令

- `ps`命令用来查看当前系统运行的进程快照，`ps aux`用来查看所有的进程，并列出进程命令；

- 常用使用`ps`命令与管道符和`grep`使用查看具体的进程是否运行，另外查看进程也可以用`ps -elf`命令：

  ![ps](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/ps.png)

- `ps`命令可以查看到进程的PID，PID可以用来杀死一个进程，使用`kill [pid]`命令杀死进程；

- 使用`ls -l /proc/[pid]`可以查看进程运行的信息和相关文件：

  ```bash
  [root@evobot ~]# ls -l /proc/447/
  总用量 0
  dr-xr-xr-x 2 root root 0 4月  19 00:35 attr
  -rw-r--r-- 1 root root 0 5月   7 22:47 autogroup
  -r-------- 1 root root 0 5月   7 22:47 auxv
  -r--r--r-- 1 root root 0 4月  19 00:35 cgroup
  --w------- 1 root root 0 5月   7 22:47 clear_refs
  -r--r--r-- 1 root root 0 4月  19 00:35 cmdline
  -rw-r--r-- 1 root root 0 4月  19 00:35 comm
  -rw-r--r-- 1 root root 0 5月   7 22:47 coredump_filter
  -r--r--r-- 1 root root 0 5月   7 22:47 cpuset
  lrwxrwxrwx 1 root root 0 5月   7 22:47 cwd -> /
  -r-------- 1 root root 0 5月   7 22:38 environ
  lrwxrwxrwx 1 root root 0 4月  19 00:35 exe -> /usr/sbin/auditd
  ```

- `ps`命令所列的信息中，包括CPU和内存百分比，`VSZ`表示进程占用的虚拟内存，`RSS`表示进程占用的物理内存与`top`命令的`RES`列相同，`TTY`表示进程在哪个tty所运行，`START`表示进程开始运行的时间，`TIME`表示进程运行的总时间，`COMMAND`表示进程的命令，`STAT`表示进程的状态，具体存在的状态如下表：

| 状态表示字符 | 说明                                     |
| :----: | -------------------------------------- |
|   D    | 不能中断的进程                                |
|   R    | run状态正在运行的进程                           |
|   S    | sleep睡眠状态的进程，有些进程运行时占用CPU时间非常短暂，也会显示为S |
|   T    | 暂停的进程，如ctrl+z暂停的进程                     |
|   +    | 表示前台进程                                 |
|   Z    | 僵尸进程                                   |
|   <    | 高优先级进程，有限占用CPU资源                       |
|   N    | 低优先级进程                                 |
|   L    | 内存中被锁了内存分页的进程，少见                       |
|   s    | 主进程，例如nginx进程，存在一个主进程，另外还衍生出多个子进程      |
|   l    | 多线程进程，一个进程可以有多个线程，线程间共享进程的内存           |

---

# 网络监控和抓包

## **netstat**命令

- `netstat`命令用来查看系统的网络状态，常用的用法分为以下三种：

  1. `netstat -lnp`：查看服务监听的端口，如下图，tcp和udp分别监听的端口，另外也列出了系统的socket文件：

  ![netstat -lnp](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/netstat-lnp.png)

  2. `netstat -an`：查看系统的网络连接状态，具体状态含义可以查看[tcp的三次握手和四次挥手](http://www.doc88.com/p-9913773324388.html)：

  ![netstat -an](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/netstat-an.png)

  3. 上面的命令输出中将TCP和UDP都列出来了，如果想要只查看tcp连接，可以使用`-t`参数，查看udp连接使用`-u`参数，使用这两个参数时不会列出socket：

  ```bash
  [root@evobot ~]# netstat -ant
  Active Internet connections (servers and established)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State      
  tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
  tcp        0      0 0.0.0.0:2233            0.0.0.0:*               LISTEN     
  tcp        0     36 10.139.151.2:2233       118.113.205.177:25473   ESTABLISHED

  [root@evobot ~]# netstat -anu
  Active Internet connections (servers and established)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State      
  udp        0      0 10.139.151.2:34290      114.114.114.114:53      ESTABLISHED
  udp        0      0 10.139.151.2:123        0.0.0.0:*                          
  udp        0      0 127.0.0.1:123           0.0.0.0:*                          
  udp        0      0 0.0.0.0:123             0.0.0.0:*                          
  udp6       0      0 :::123                  :::*                              
  ```

- `ss -an`命令同样也可以输出系统的网络连接状态，但不会显示进程名：

  ```bash
  [root@evobot ~]# ss -an
  Netid  State      Recv-Q Send-Q       Local Address:Port                      Peer Address:Port              
  nl     UNCONN     0      0                        0:490                                   *                   
  u_str  ESTAB      0      0      /run/systemd/journal/stdout 12258                                * 12257              
  udp    ESTAB      0      0             10.139.151.2:35758                  114.114.114.114:53                 
  udp    UNCONN     0      0             10.139.151.2:123                                  *:*                  
  udp    UNCONN     0      0                127.0.0.1:123                                  *:*                  
  tcp    LISTEN     0      128                      *:111                                  *:*                  
  tcp    LISTEN     0      128                      *:2233                                 *:*                  
  tcp    ESTAB      0      0             10.139.151.2:2233                   118.113.205.177:25473              
  ```

- `netstat -an | awk '/^tcp/{++sta[$NF]} END {for(key in sta) print key,"\t",sta[key]}'`命令可以统计出系统各个网络连接的总数：

  ```bash
  [root@evobot ~]# netstat -an | awk '/^tcp/{++sta[$NF]} END {for(key in sta) print key,"\t",sta[key]}'
  LISTEN 	 2
  ESTABLISHED 	 1
  ```

  - 这其中我们主要关注**ESTABLISHED**状态数，其表示正在和服务器通信的连接数。

## **tcpdump**命令

- `tcpdump`命令是用来网络抓包的命令行工具，使用前需要安装`tcpdump`软件包；

- 常用用法有以下几种：

  1. `tcpdump -nn -i [网卡名]`： 这里的参数`-nn`，第一个**n**表示显示ip地址，否则显示主机名，第二个**n**表示显示端口，否则使用服务名表示；

  ![tcpdump -nn](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/tcpdump-nn.png)

  ![tcpdump -i](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/tcpdum-i.png)

  > 上面的输出中分别表示时间，IP，源ip.端口，目的ip.端口以及数据包信息；

  2. `tcpdump -nn port [端口号]`：抓取指定端口的数据包，同样也可以使用`tcpdump -nn -i eth0 not port [端口号]`抓取除指定端口外的数据包，或者`tcpdump -nn -i eth0 not port [端口号] and host [ip地址]`来抓取指定host除指定端口外的数据包：

  ```bash
  root@Deppin:~# tcpdump -nn -i wlp2s0 not port 22 and host 180.97.33.108
  tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
  listening on wlp2s0, link-type EN10MB (Ethernet), capture size 262144 bytes
  00:23:25.460925 IP 192.168.199.199.43109 > 180.97.33.108.80: Flags [F.], seq 836106603, ack 3946427568, win 229, length 0
  00:23:29.332936 IP 192.168.199.199.43109 > 180.97.33.108.80: Flags [F.], seq 0, ack 1, win 229, length 0
  00:23:29.332966 IP 192.168.199.199.34253 > 180.97.33.108.80: Flags [F.], seq 3073252526, ack 1107915791, win 229, length 0
  00:23:29.332975 IP 192.168.199.199.42371 > 180.97.33.108.80: Flags [F.], seq 1787139822, ack 537241163, win 229, length 0
  ^C
  4 packets captured
  4 packets received by filter
  0 packets dropped by kernel
  ```

  3. 也可以指定抓取的数据包个数和指定存储到哪个文件：`root@Deppin:~# tcpdump -nn -i wlp2s0 -c 5 -w /tmp/1.cap`:

  ```bash
  root@Deppin:~# tcpdump -nn -i wlp2s0 -c 5 -w /tmp/1.cap
  tcpdump: listening on wlp2s0, link-type EN10MB (Ethernet), capture size 262144 bytes
  5 packets captured
  35 packets received by filter
  0 packets dropped by kernel

  root@Deppin:~# file /tmp/1.cap 
  /tmp/1.cap: tcpdump capture file (little-endian) - version 2.4 (Ethernet, capture length 262144)
  ```

  {% note success %} 
  抓取的文件是无法使用cat命令查看的，查看需要使用`tcpdump -r [filename]`进行查看。
  {% endnote %}

- `tshark`命令是类似`tcpdump`的工具，使用这个命令需要安装`wireshark`软件包，常用用法如下:

  ```bash
  # tshark -n -t a -R http.request -T fields -e "frame.time" -e "ip.src" -e "http.host" -e "http.request.method" -e "http.request.uri"
  tshark: -R without -2 is deprecated. For single-pass filtering use -Y.
  Running as user "root" and group "root". This could be dangerous.
  Capturing on 'eth0'
  "May  8, 2018 00:35:03.807043090 CST"	10.139.151.2	receiver.barad.tencentyun.com	POST	/ca_report.cgi
  "May  8, 2018 00:35:08.814790915 CST"	10.139.151.2	receiver.barad.tencentyun.com	POST	/ca_report.cgi
  "May  8, 2018 00:35:13.823298617 CST"	10.139.151.2	receiver.barad.tencentyun.com	POST	/heart_report.cgi
  "May  8, 2018 00:35:18.831342319 CST"	10.139.151.2	receiver.barad.tencentyun.com	POST	/ca_report.cgi
  "May  8, 2018 00:35:23.840005021 CST"	10.139.151.2	receiver.barad.tencentyun.com	POST	/ca_report.cgi
  ^C1 packet dropped
  5 packets captured
  ```

  {% note success %} 
  上面的命令可以看到详细的ip地址、访问的网址，请求方法和请求的资源。关于`tshark`更多的用法，可以参考[tshark----wireshark的命令行工具](https://blog.csdn.net/hello2mao/article/details/53967900)
  {% endnote %}
---