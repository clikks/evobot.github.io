---
title: Linux日常运维(一)—系统资源监控
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 707efc0b
date: 2018-05-05 01:01:16
image:
---

本文主要介绍系统管理相关的w、vmstat、top、sar、nload几个命令和它们的使用方法，这几个命令主要用来监控系统状态，如CPU使用率，系统负载，内存和进程状态等。

<!--more-->

---

# 监控系统状态

## **w**命令

- `w`命令的输出如下，主要分为三个部分：

  ```bash
  [root@evobot ~]# w
   01:10:04 up 16 days, 34 min,  1 user,  load average: 0.00, 0.01, 0.05
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  evobot   pts/0    192.168.75.254   00:58    4.00s  0.02s  0.00s sshd: evobot [priv]
  ```

  - 第一行首先是系统时间和系统启动时间，这里是16天34分钟，然后是登录的用户数，最后一段是系统负载，有三个数字，分别表示**1分钟**、**5分钟**、**15分钟**时间段内，系统的负载值，负载值是单位时间内，使用CPU的进程数的平均值，系统负载的最佳状态是一分钟内与逻辑CPU个数相等的负载值；

  - 查看逻辑CPU个数，使用命令`cat /proc/cpuinfo`，查看输出的**processor**的个数是多少，那么CPU个数就是多少：

    ```bash
    [root@evobot ~]# cat /proc/cpuinfo
    processor	: 0		# 只有一个，说明CPU为一核
    vendor_id	: GenuineIntel
    cpu family	: 6
    model		: 79
    model name	: Intel(R) Xeon(R) CPU E5-26xx v4
    stepping	: 1
    microcode	: 0x1
    cpu MHz		: 2394.446
    cache size	: 4096 KB
    ```

  - 如果是四核CPU，那么一分钟负载最佳数值就是4，大于4说明系统负载较大，小于4说明系统比较空闲；

  - `w`命令的第一行输出与`uptime`命令的输出相同，所以也可以使用`uptime`命令查看CPU负载：

    ```bash
    [root@evobot ~]# uptime
     01:37:02 up 16 days,  1:01,  1 user,  load average: 0.00, 0.01, 0.05
    ```

  - 第二行和第三行显示当前登录的用户的一些信息，分别显示用户名，登录的tty，网络远程登录tty都是`pts/[n]`，n是一个数字，本地登录则显示`tty[n]`然后是登录用户的IP地址，登录的时间，登录用户空闲时间以及最后使用CPU的时间和运行的命令。


## **vmstat**命令

- `vmstat`命令可以查看CPU，内存，swap交换分区，磁盘的io，和系统进程等相关信息；

- 常用的`vmstat`命令方式为`vmstat 1`，这样可以持续输出系统信息，并且也可以指定输出次数，如输出5次之后结束命令，则用法为`vmstat 1 5`：

  ```bash
  [root@evobot ~]# vmstat 1 5
  procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
   r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
   2  0      0 512080 226420 1047608    0    0     0    19   19   27  1  0 99  0  0
   0  0      0 512080 226420 1047608    0    0     0     0   83   88  0  0 100  0  0
   0  0      0 512096 226420 1047608    0    0     0     0   55   53  0  0 100  0  0
   1  0      0 512096 226420 1047608    0    0     0     0   63   66  0  0 100  0  0
   0  0      0 512096 226420 1047608    0    0     0     0   67   70  0  0 100  0  0
  ```

- `vmstat`命令的输出中，主要关注以下几列：

<style>
table th {
    text-align: center;
}
table th:first-of-type {
    width: 120px;
}
</style>

|    常用列    | 输出说明                                     |
| :-------: | ---------------------------------------- |
|    `r`    | 表示有多少个进程处于`run`运行状态，`run`状态的进程包括正在运行和排队等待CPU资源的进程,值最大与CPU核数相同 |
|    `b`    | 表示被CPU以外资源如硬盘，网络等阻塞(block)的进程，进程等待外部资源处理，大于1需要关注    |
|  `swapd`  | 系统使用交换空间的大小，若这一列的数值在持续变化，说明系统内存不足     |
| `si & so` | `si`表示有多少块(KB)数据从swap进入内存，`so`表示有多少块数据从内存出来到swap中去 |
| `bi & bo` | `bi`表示有多少数据从磁盘读取到内存中，`bo`表示有多少数据写入的磁盘，当`bi`、`bo` 过高时，`b`列的值也会增加，说明进程等待磁盘io被阻塞 |
|   `us`    | 用户态占用CPU资源百分比，当数值大于50时，说明CPU资源不足  |
|   `sy`    | 系统本身的进程占用CPU资源百分比                        |
|   `id`    | CPU空闲资源比例，`us+sy+id`为100%                |
|   `wa`    | 等待CPU资源的进程百分比                            |

- `vmstat`主要用来判断系统的瓶颈所在，如CPU资源不足，内存资源不足等等。

## **top**命令

- top命令类似于windows中的进程管理器，可以查看系统具体运行的进程，默认运行`top`，命令输出为3秒钟刷新一次：

  ![top](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/top_cmd.png)

  ```bash
  Tasks: 152 total,   1 running, 151 sleeping,   0 stopped,   0 zombie
  ```

  - 第二行显示了系统目前的进程数、正在运行的进程数和休眠的进程数，另外还会显示已经停止的进程和僵尸进程的个数，僵尸进程是指父进程因为意外等原因被终止，子进程处于无法管理的状态的进程；

  ```bash
  %Cpu(s):  0.0 us,  0.1 sy,  0.0 ni, 99.9 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
  ```

  - 这一行显示的CPU占用率，类似`vmstat`命令的输出，其中`st`表示被偷走的CPU百分比，例如服务器上运行的虚拟机，可能会偷走部分CPU资源；
  - CPU百分比中`us`和系统负载不一定相关，负载中的进程可能存在占用CPU等待的情况，所以`us`会比较低，而`us`占用高，会导致其他进程排队，从而造成系统负载高；

  ```bash
  KiB Mem :  3928092 total,  2185976 free,   294268 used,  1447848 buff/cache
  KiB Swap:  4075516 total,  4075516 free,        0 used.  3274380 avail Mem 
  ```

  - 这两行输出的内容是内存和swap交换分区使用情况，主要关注`free`剩余空闲内存。

  ```bash
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                       
   6717 evobot    20   0   42260   3844   3252 R   0.3  0.1   0:01.91 top       
  ```

  - `top`命令剩余的显示内容就是系统的进程，默认按照进程占用CPU百分比进行排序，每列所表示的信息如下：

|     列名      |    所表示信息     |
| :---------: | :----------: |
|    `PID`    |     进程ID     |
| `PR` & `NI` |   进程优先级信息    |
|    `RES`    | 物理内存使用大小(KB) |
|   `%CPU`    |    CPU使用率    |
|   `%MEM`    |    内存使用率     |
|  `COMMAND`  |     进程命令     |

- `top`命令默认按照CPU使用率排序，可以键入`M`，则会按照内存使用率排序，`P`则是按照CPU使用率排序，按`q`退出top命令，按`1`则会将CPU显示单个CPU内核的占用率：

  ```bash
  %Cpu0  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
  %Cpu1  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
  %Cpu2  :  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
  %Cpu3  :  0.7 us,  0.0 sy,  0.0 ni, 99.3 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
  ```

- 使用`top -c`则top会显示进程的具体命令：

  ![top-c](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/top-c.png)      

- 而`top -bn1`则是静态的将所有进程列出来，适合在shell脚本中使用:

  ![top-bn1](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/top-bn1.png)

---

# 监控网卡状态

## **sar**命令

- `sar`命令如果未安装，需要安装`sysstat`软件包：

  ```bash
  yum install -y sysstat
  ```

- 直接运行`sar`命令会提示*无法打开 /var/log/sa/sa05: 没有那个文件或目录*，这是因为`sar`命令每隔十分钟会把系统状态过滤并保存在上面的目录下，`/var/log/sa/sa[n]`这里的n是以生成文件的日期的日命名，文件最多保留一个月；

- `sar -n DEV 1 10`用法，表示输出网卡的信息，`1 10`表示一秒输出一次，输出10次，与`vmstat 1 5`的用法类似：

  ![sar-n](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/sar-n.png)

  - 图片的输出中，第一列是时间，第二列是网卡名，主要关注下面这几列：`rxpck/s`是每秒接收到数据包，单位是个，`txpck/s`是发送的数据表，`rxkB/s`和`txkB/s`则分别表示每秒接收和发送的数据量，单位为KB。
  - 实际运维中，需要关注接收的数据包和接收的数据量，如果数据过高，可能服务器受到了攻击，另外也要关注网卡的带宽是否被跑满；

- 使用命令`sar -n DEV -f /var/log/sa/sa[n]`可以查看指定日期的历史数据：

  ```bash
  [root@evobot ~]# sar -n DEV -f /var/log/sa/sa05 
  Linux 3.10.0-693.21.1.el7.x86_64 (evobot) 	2018年05月05日 	_x86_64_	(1 CPU)

  03时20分01秒     IFACE   rxpck/s   txpck/s    rxkB/s    txkB/s   rxcmp/s   txcmp/s  rxmcst/s
  03时30分01秒      eth0      2.72      2.87      0.26      0.36      0.00      0.00      0.00
  03时30分01秒        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
  平均时间:      eth0      2.72      2.87      0.26      0.36      0.00      0.00      0.00
  平均时间:        lo      0.00      0.00      0.00      0.00      0.00      0.00      0.00
  ```

- `sar -q 1 10`可以查看系统的负载，输出与`w`命令的系统负载相同，一般不会使用sar命令每秒输出查看系统负载，而是使用`sar -q -f /var/log/sa/sa[n]`查看当天历史负载，也可以指定sar生成的日志文件查看之前时间的历史负载：

  ```bash
  [root@evobot ~]# sar -q
  Linux 3.10.0-693.21.1.el7.x86_64 (evobot) 	2018年05月05日 	_x86_64_	(1 CPU)

  03时20分01秒   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
  03时30分01秒         2        90      0.00      0.01      0.05         0
  平均时间:         2        90      0.00      0.01      0.05         0

  [root@evobot ~]# sar -q -f /var/log/sa/sa05 
  Linux 3.10.0-693.21.1.el7.x86_64 (evobot) 	2018年05月05日 	_x86_64_	(1 CPU)

  03时20分01秒   runq-sz  plist-sz   ldavg-1   ldavg-5  ldavg-15   blocked
  03时30分01秒         2        90      0.00      0.01      0.05         0
  03时40分01秒         2        89      0.00      0.01      0.05         0
  平均时间:         2        90      0.00      0.01      0.05         0
  ```

- `sar -b`可以查看磁盘的读写信息：

  ```bash
  [root@evobot ~]# sar -b
  Linux 3.10.0-693.21.1.el7.x86_64 (evobot) 	2018年05月05日 	_x86_64_	(1 CPU)

  03时20分01秒       tps      rtps      wtps   bread/s   bwrtn/s
  03时30分01秒      2.33      0.01      2.32      0.15     45.55
  03时40分01秒      1.79      0.00      1.79      0.00     23.11
  平均时间:      2.06      0.01      2.05      0.07     34.33

  [root@evobot ~]# sar -b 1 5
  Linux 3.10.0-693.21.1.el7.x86_64 (evobot) 	2018年05月05日 	_x86_64_	(1 CPU)

  03时41分26秒       tps      rtps      wtps   bread/s   bwrtn/s
  03时41分27秒      0.00      0.00      0.00      0.00      0.00
  03时41分28秒      0.00      0.00      0.00      0.00      0.00
  03时41分29秒      0.00      0.00      0.00      0.00      0.00
  03时41分30秒      0.00      0.00      0.00      0.00      0.00
  03时41分31秒      1.98      0.00      1.98      0.00     31.68
  平均时间:      0.40      0.00      0.40      0.00      6.40

  [root@evobot ~]# sar -b -f /var/log/sa/sa05 
  Linux 3.10.0-693.21.1.el7.x86_64 (evobot) 	2018年05月05日 	_x86_64_	(1 CPU)

  03时20分01秒       tps      rtps      wtps   bread/s   bwrtn/s
  03时30分01秒      2.33      0.01      2.32      0.15     45.55
  03时40分01秒      1.79      0.00      1.79      0.00     23.11
  平均时间:      2.06      0.01      2.05      0.07     34.33
  ```

- `sar`命令在/var/log/sa/目录下除了生成sa[n]文件外，在第二天还会生成sar[n]文件，两个文件的区别是sa[n]文件为二进制文件，而sar[n]则是文本文件，可以使用cat命令查看 。

## **nload**命令

- `nload`默认也是没有安装，需要安装`epel-release`扩展源再安装`nload`软件包；

- 执行`nload`命令，可以动态的显示网卡信息：

  ![nload](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/nload.png)

  - 输出界面中第一行会显示网卡的名字、ip以及当前是第几张网卡，使用向下方向键可以切换网卡，`q`键退出；
  - 界面上半部分显示的是进入的流量，下半部分是出口流量，如果进入的流量非常大，可能服务器受到了攻击，右下角显示的信息分别是当前网络速率，平均、最小、最大速率；

---
