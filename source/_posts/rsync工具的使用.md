---
title: rsync工具的使用
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: d9f5c2d9
date: 2018-05-14 21:31:20
image:
---



本文介绍了LInux文件同步工具——rsync的用法，和一些相关的配置，以及如何使用rsync通过ssh或者服务进行同步。

<!--more-->

---

# rsync同步工具

## rsync介绍

- 在Linux中，经常会需要对某个目录进行备份到其他机器或本机的其他目录，如备份到本机目录，如果使用cp命令，当文件在持续增加并且增加的内容很少时，由于cp命令拷贝时可以覆盖之前的文件，就导致实际操作效率不高，资源占用也较大，所以使用rsync命令可以有效的解决这些问题；

- rsync可以实现增量拷贝，只备份新增的部分，而且支持远程同步备份；

- `rsync -av [/path/file] [/path/file]`命令可以将文件同步到指定的目录下并且改名：

  ```bash
  [root@www ~]# rsync -av /etc/passwd ~/2.passwd
  sending incremental file list
  passwd

  sent 1,145 bytes  received 35 bytes  2,360.00 bytes/sec
  total size is 1,053  speedup is 0.89
  [root@www ~]# ls
  1.passwd
  [root@www ~]# head -n3 2.passwd 
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  ```

- 而远程同步，命令格式为`rsync -av [/path/filename] user@host:/path/filename`:

  ```bash
  [root@www ~]# rsync -av /etc/passwd root@192.168.199.100:/tmp/rsync.passwd
  root@192.168.199.100's password: 
  sending incremental file list
  passwd

  sent 1,139 bytes  received 34 bytes  67.03 bytes/sec
  total size is 1,053  speedup is 0.90
  ```

  > 使用rsync远程备份需要远端机器上有rsync命令

- rsync命令还有其他几种用法：

  - `rsync [OPTION] ... SRC DEST`：OPTION选项指`-av`，然后SRC源文件或目录，DEST目的文件或目录；
  - `rsync [OPTION] ... SRC [user]@host:DEST`：这里的用法就是上面的远程备份用法，这里的`user@`可以省略不写，省略时，默认以当前终端用户去连接远程机器；
  - `rsync [OPTION] ... [user]@host:SRC DEST`：这里使用远程机器的文件路径作为SRC源文件，可以讲远程文件同步到本地上来；
  - `rsync [OPTION] ... SRC [user]@host::DEST`：需要注意这里远程机器的host后面是两个`:`，通过服务将本地源文件同步到远程机器；
  - `rsync [OPTION] ... [user]@host::SRC DEST`：与上面相同，使用两个`:`，通过服务将远程源文件同步到本地。

## rsync常用选项

|   常用选项    |                    作用                    |
| :-------: | :--------------------------------------: |
|    -a     |          综合选项，包含`-rtplgoD`这几个选项          |
|    -r     |              同步目录，类似cp的-r选项              |
|    -v     |                同步时显示同步过程                 |
|    -l     |                  保留软连接                   |
|    -L     |             同步软连接时将源文件同样同步过去             |
|    -p     |                保持文件的权限属性                 |
|    -o     |                 保持文件的属主                  |
|    -g     |                 保持文件的属组                  |
|    -D     |                 保持设备文件信息                 |
|    -t     |                保持文件的时间属性                 |
| --delete  |             删除目的目录中源目录没有的文件              |
| --exclude | 过滤指定文件，如--exclude "logs"会将文件名包含logs的文件或目录过滤，不同步，支持通配 |
|    -P     |             显示同步过程，如速率，比-v详细             |
|    -u     |        使用此选项后，若DEST中文件比SRC新，则不同步         |
|    -z     |                 传输时进行压缩                  |

- 示例

```bash
[root@www ~]# ls -l rsync_dir/test 
lrwxrwxrwx 1 root root 9 5月  14 23:10 rsync_dir/test -> /tmp/test
[root@www ~]# rsync -avL rsync_dir/ /tmp/rsync_dest
sending incremental file list
created directory /tmp/rsync_dest
./
111
222
asnd
test
333/

sent 318 bytes  received 137 bytes  910.00 bytes/sec
total size is 0  speedup is 0.00
[root@www ~]# ls -l /tmp/rsync_dest/
总用量 0
-rw-r--r-- 1 root root 0 5月  14 23:10 111
-rw-r--r-- 1 root root 0 5月  14 23:10 222
drwxr-xr-x 2 root root 6 5月  14 23:10 333
-rw-r--r-- 1 root root 0 5月  14 23:10 asnd
-rw-r--r-- 1 root root 0 5月  14 23:10 test
```

> 使用`-L`之后，软连接文件同步到目的目录变成了普通文件

```bash
[root@www ~]# ls /tmp/rsync_dest/
111  222  333  asnd  new.txt  test
[root@www ~]# ls rsync_dir/
111  222  333  asnd  test
[root@www ~]# rsync -av --delete rsync_dir/ /tmp/rsync_dest/
sending incremental file list
deleting new.txt
./
test -> /tmp/test

sent 177 bytes  received 34 bytes  422.00 bytes/sec
total size is 9  speedup is 0.04
[root@www ~]# ls /tmp/rsync_dest/
111  222  333  asnd  test
```

> 使用`--delete`选项，会将目的目录中与源目录不一致的文件删除。

```bash
[root@www rsync_dir]# ls
111.txt  222  333  asnd.txt  test
[root@www rsync_dir]# rsync -av --exclude "*.txt" /root/rsync_dir/ /tmp/dest/
sending incremental file list
created directory /tmp/dest
./
222
test -> /tmp/test
333/

sent 179 bytes  received 77 bytes  512.00 bytes/sec
total size is 9  speedup is 0.04
[root@www rsync_dir]# ls /tmp/dest/
222  333  test
```

> 使用`--exclude`选项可以过滤掉不想同步的文件，同时可以指定多个`--exclude`进行多种过滤。

```bash
[root@www rsync_dir]# rsync -avP /root/rsync_dir/ /tmp/dest
sending incremental file list
created directory /tmp/dest
./
111.txt
              0 100%    0.00kB/s    0:00:00 (xfr#1, to-chk=5/7)
222
              0 100%    0.00kB/s    0:00:00 (xfr#2, to-chk=4/7)
asnd.txt
              0 100%    0.00kB/s    0:00:00 (xfr#3, to-chk=3/7)
hello.txt
    104,857,600 100%  168.87MB/s    0:00:00 (xfr#4, to-chk=2/7)
test -> /tmp/test
333/

sent 104,883,565 bytes  received 138 bytes  69,922,468.67 bytes/sec
total size is 104,857,609  speedup is 1.00
```

> -P选项会将同步时的详细过程打印出来

```bash
[root@www rsync_dir]# cat /tmp/dest/111.txt 
asdniainaifnai
[root@www rsync_dir]# cat 111.txt 
[root@www rsync_dir]# rsync -avPu /root/rsync_dir/ /tmp/dest/
sending incremental file list
./

sent 207 bytes  received 20 bytes  454.00 bytes/sec
total size is 104,857,609  speedup is 461,927.79
[root@www rsync_dir]# cat 111.txt 
[root@www rsync_dir]# cat /tmp/dest/111.txt 
asdniainaifnai
```

> -u选项能够保证目标目录下更新的文件不被源文件所覆盖。

## 通过ssh同步

- rsync通过ssh同步，命令的形式为`rsync -av SRC [user]@host:DEST`；
- 如果对方机器或者本地的ssh端口不是默认的22端口，则命令的形式为`rsync -av -e "ssh -p 2022" SRC [user]@host:DEST`；

---

