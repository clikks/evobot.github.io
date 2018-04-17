---
title: Linux磁盘管理——df & du命令及磁盘分区
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 43eb8ab5
date: 2018-04-09 20:30:03
image:
---
![磁盘分区](http://p5qynomrl.bkt.clouddn.com/1523288634264cki496jp.png?imageslim)
主要介绍Linux系统中磁盘管理命令`df`和`du`命令的用法，以及使用`fdisk`和`parted`命令行工具对磁盘的分区操作。

<!--more-->

# 磁盘管理

## df命令

- `df`命令是用来查看文件系统磁盘使用情况的命令，直接执行`df`命令得到如下结果：

  ```bash
  [root@evobot ~]# df
  文件系统          1K-块    已用     可用 已用% 挂载点
  /dev/vda1      51474024 2633324 46534624    6% /
  devtmpfs         932260       0   932260    0% /dev
  tmpfs            941768      24   941744    1% /dev/shm
  tmpfs            941768     356   941412    1% /run
  tmpfs            941768       0   941768    0% /sys/fs/cgroup
  tmpfs            188356       0   188356    0% /run/user/0
  tmpfs            188356       0   188356    0% /run/user/1001
  ```

- 命令执行的结果中，第一列表示文件系统，也就是磁盘分区的名字；第二列为磁盘总大小，单位为KB；第三列为分区已使用大小；第四列为磁盘剩余空间大小；第五列为磁盘已使用空间大小百分比；最后一列为磁盘挂载点，即系统内的目录。

- `df`命令常用选项`-h`可以人性化显示，如磁盘空间大小单位：

  ```bash
  [root@evobot ~]# df -h
  文件系统        容量  已用  可用 已用% 挂载点
  /dev/vda1        50G  2.6G   45G    6% /
  devtmpfs        911M     0  911M    0% /dev
  tmpfs           920M   24K  920M    1% /dev/shm
  tmpfs           920M  356K  920M    1% /run
  tmpfs           920M     0  920M    0% /sys/fs/cgroup
  tmpfs           184M     0  184M    0% /run/user/0
  tmpfs           184M     0  184M    0% /run/user/1001

  ```

- 在第一列的文件系统中，`tmpfs`代表临时文件系统，相应挂载点内的文件会在重启后消失，而挂载点的`/dev/shm`则是内存，大小为实际内存大小的一半；

- 而分区时指定的`swap`分区并不会在`df`命令中列出来，使用`free`命令可以查看`swap`分区大小:

  ```bash
  [root@evobot ~]# free
                total        used        free      shared  buff/cache   available
  Mem:        1883540       87236      575564         380     1220740     1598584
  Swap:             0           0           0

  ```

- `df`的另一个常用选项为`-i`，用来查看分区的`inode`使用情况，`inode`的多少与磁盘分区大小有关，并且是在分区格式化时就分配的：

  ```bash
  [root@evobot ~]# df -i
  文件系统         Inode 已用(I) 可用(I) 已用(I)% 挂载点
  /dev/vda1      3276800   81696 3195104       3% /
  devtmpfs        233065     320  232745       1% /dev
  tmpfs           235442       7  235435       1% /dev/shm
  tmpfs           235442     376  235066       1% /run
  tmpfs           235442      16  235426       1% /sys/fs/cgroup
  tmpfs           235442       1  235441       1% /run/user/0
  tmpfs           235442       1  235441       1% /run/user/1001

  ```

## du命令

- du命令用来查看一个文件或者目录的大小，一般常用用法为`du -sh (文件或目录)`：

  ```bash
  [root@evobot ~]# du -sh source/
  260K	source/

  [root@evobot ~]# du -sh /etc/passwd
  4.0K	/etc/passwd
  [root@evobot ~]# ls -lh /etc/passwd
  -rw-r--r-- 1 root root 1.4K 4月   3 23:31 /etc/passwd

  ```

- 上面的命令结果中，同一个文件，使用`du`命令和`ls`命令查看但文件大小却不相同，这是因为磁盘分区时是由一个个4K大小的块组成的，当文件小于4K时，也会占用磁盘分区上一个块，所以使用`du`命令查看时，会显示为4K；

- `du`命令在不加选项时，默认不显示单位，并且会将目录下的所有文件都列出来；

---

# 磁盘分区

- 在实际使用中，经常需要给服务器磁盘进行扩容，增加磁盘，在虚拟机上，我们直接为系统增加一块硬盘，由于虚拟机并不支持自动识别新硬盘，所以需要重启之后才能查看到新添加的磁盘。



## **fdisk**命令

- `fdisk `命令可以用来查看系统中的磁盘，也可以对磁盘进行分区操作，其中查看磁盘的选项为`fdisk -l`：

  ```bash
  [root@localhost ~]# fdisk -l

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节



  磁盘 /dev/sda：32.2 GB, 32212254720 字节，62914560 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0x00022876

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sda1   *        2048      411647      204800   83  Linux
  /dev/sda2          411648     4605951     2097152   82  Linux swap / Solaris
  /dev/sda3         4605952    62914559    29154304   83  Linux

  ```

- 上面的`/dev/sdb`就是新增加的磁盘，使用`fdisk (磁盘名)`进行磁盘管理，进入`fdisk`的命令行界面后，可以输入`m`回车查看帮助信息：

  ```bash
  [root@localhost ~]# fdisk /dev/sdb
  欢迎使用 fdisk (util-linux 2.23.2)。

  更改将停留在内存中，直到您决定将更改写入磁盘。
  使用写入命令前请三思。

  Device does not contain a recognized partition table
  使用磁盘标识符 0x216a8918 创建新的 DOS 磁盘标签。

  命令(输入 m 获取帮助)：m
  命令操作
     a   toggle a bootable flag
     b   edit bsd disklabel
     c   toggle the dos compatibility flag
     d   delete a partition
     g   create a new empty GPT partition table
     G   create an IRIX (SGI) partition table
     l   list known partition types
     m   print this menu
     n   add a new partition
     o   create a new empty DOS partition table
     p   print the partition table
     q   quit without saving changes
     s   create a new empty Sun disklabel
     t   change a partition's system id
     u   change display/entry units
     v   verify the partition table
     w   write table to disk and exit
     x   extra functionality (experts only)

  命令(输入 m 获取帮助)：

  ```

- 常用的操作为`n`增加新分区，`p`打印分区信息，`w`保存分区信息并退出、`d`删除分区。

## 使用fdisk对磁盘分区

- 使用`ｐ`命令查看当前磁盘分区情况：

  ```bash
  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0x216a8918

     设备 Boot      Start         End      Blocks   Id  System

  ```

- 当前磁盘没有分区，使用`n`命令增加一个分区：

  ```bash
  命令(输入 m 获取帮助)：n
  Partition type:
     p   primary (0 primary, 0 extended, 4 free)
     e   extended
  Select (default p):
  ```

- 由于`fdisk`只能对2T以下大小的`mbr`分区格式的磁盘进行分区，而`mbr`格式的磁盘主分区及扩展分区总数不能超过4个，所以`n`命令之后给出的选项，`p`为主分区，`e`为扩展分区：

  ```bash
  Select (default p): p
  分区号 (1-4，默认 1)：1
  起始 扇区 (2048-20971519，默认为 2048)：
  将使用默认值 2048
  Last 扇区, +扇区 or +size{K,M,G} (2048-20971519，默认为 20971519)：+2G
  分区 1 已设置为 Linux 类型，大小设为 2 GiB

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0x216a8918

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     4196351     2097152   83  Linux
  ```

- 上面的步骤创建了一个主分区，指定了分区号为1，大小使用`+2G`的形式指定磁盘大小为2G，使用`p`命令可以查看新创建的分区；

- 主分区一旦达到4个，磁盘将不能够再继续分区，可以使用`d`命令删除分区后，划分一个扩展分区：

  ```bash
  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xea193fb2

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     4196351     2097152   83  Linux
  /dev/sdb2         4196352    10487807     3145728   83  Linux
  /dev/sdb3        10487808    16779263     3145728   83  Linux
  /dev/sdb4        16779264    20971519     2096128   83  Linux

  命令(输入 m 获取帮助)：n
  If you want to create more than four partitions, you must replace a
  primary partition with an extended partition first.
  # 划分4个主分区后，再新增分区会有相应的提示

  命令(输入 m 获取帮助)：d
  分区号 (1-3，默认 3)：3
  分区 3 已删除

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xea193fb2

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     4196351     2097152   83  Linux
  /dev/sdb2         4196352    10487807     3145728   83  Linux
  # 已经将3,4两个分区删除

  命令(输入 m 获取帮助)：n
  Partition type:
     p   primary (2 primary, 0 extended, 2 free)
     e   extended
  Select (default p): e
  分区号 (3,4，默认 3)：
  起始 扇区 (10487808-20971519，默认为 10487808)：
  将使用默认值 10487808
  Last 扇区, +扇区 or +size{K,M,G} (10487808-20971519，默认为 20971519)：+3G
  分区 3 已设置为 Extended 类型，大小设为 3 GiB

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xea193fb2

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     4196351     2097152   83  Linux
  /dev/sdb2         4196352    10487807     3145728   83  Linux
  /dev/sdb3        10487808    16779263     3145728    5  Extended

  命令(输入 m 获取帮助)：n
  Partition type:
     p   primary (2 primary, 1 extended, 1 free)
     l   logical (numbered from 5)
  Select (default p): 

  ```

- 从上面的命令结果可以看到新创建的扩展分区的`id`为5，而主分区的类型`id`则为83，并且由于主分区加扩展分区未达到4个，再次增加分区时，可以继续增加主分区，也可以选择在扩展分区内添加逻辑分区，使用`l`命令增加逻辑分区：

  ```bash
  命令(输入 m 获取帮助)：n
  Partition type:
     p   primary (2 primary, 1 extended, 1 free)
     l   logical (numbered from 5)
  Select (default p): p
  已选择分区 4
  起始 扇区 (16779264-20971519，默认为 16779264)：
  将使用默认值 16779264
  Last 扇区, +扇区 or +size{K,M,G} (16779264-20971519，默认为 20971519)：
  将使用默认值 20971519
  分区 4 已设置为 Linux 类型，大小设为 2 GiB

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xea193fb2

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     4196351     2097152   83  Linux
  /dev/sdb2         4196352    10487807     3145728   83  Linux
  /dev/sdb3        10487808    16779263     3145728    5  Extended
  /dev/sdb4        16779264    20971519     2096128   83  Linux

  命令(输入 m 获取帮助)：n	#当主分区和逻辑分区达到4个时，再增加分区只会增加逻辑分区
  All primary partitions are in use
  添加逻辑分区 5
  起始 扇区 (10489856-16779263，默认为 10489856)：
  将使用默认值 10489856
  Last 扇区, +扇区 or +size{K,M,G} (10489856-16779263，默认为 16779263)：+1G
  分区 5 已设置为 Linux 类型，大小设为 1 GiB

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xea193fb2

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     4196351     2097152   83  Linux
  /dev/sdb2         4196352    10487807     3145728   83  Linux
  /dev/sdb3        10487808    16779263     3145728    5  Extended
  /dev/sdb4        16779264    20971519     2096128   83  Linux
  /dev/sdb5        10489856    12587007     1048576   83  Linux
  # 新增的分区 5 就是逻辑分区
  ```

- 逻辑分区的分区号在删除后会重新排序，并不会留空：

  ```bash
  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xea193fb2

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     4196351     2097152   83  Linux
  /dev/sdb2         4196352    10487807     3145728   83  Linux
  /dev/sdb3        10487808    16779263     3145728    5  Extended
  /dev/sdb4        16779264    20971519     2096128   83  Linux
  /dev/sdb5        10489856    12587007     1048576   83  Linux
  /dev/sdb6        12589056    16779263     2095104   83  Linux

  命令(输入 m 获取帮助)：d
  分区号 (1-6，默认 6)：5
  分区 5 已删除

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xea193fb2

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     4196351     2097152   83  Linux
  /dev/sdb2         4196352    10487807     3145728   83  Linux
  /dev/sdb3        10487808    16779263     3145728    5  Extended
  /dev/sdb4        16779264    20971519     2096128   83  Linux
  /dev/sdb5        12589056    16779263     2095104   83  Linux
  # 可以看到原来的6号分区变成了5号分区
  ```

- `fdisk`中使用`q`命令可以放弃对分区的更改并退出；

- 当划分扩展分区时，逻辑分区的分区号一定是从5开始的，前4个分区号是留给主分区或扩展分区的：

  ```bash
  命令(输入 m 获取帮助)：n
  Partition type:
     p   primary (0 primary, 0 extended, 4 free)
     e   extended
  Select (default p): e	#先创建一个逻辑分区
  分区号 (1-4，默认 1)：
  起始 扇区 (2048-20971519，默认为 2048)： 
  将使用默认值 2048
  Last 扇区, +扇区 or +size{K,M,G} (2048-20971519，默认为 20971519)：+3G
  分区 1 已设置为 Extended 类型，大小设为 3 GiB

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xab7f9d31

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     6293503     3145728    5  Extended

  命令(输入 m 获取帮助)：n	# 再创建一个分区号为3的主分区
  Partition type:
     p   primary (0 primary, 1 extended, 3 free)
     l   logical (numbered from 5)
  Select (default p): p
  分区号 (2-4，默认 2)：3
  起始 扇区 (6293504-20971519，默认为 6293504)：
  将使用默认值 6293504
  Last 扇区, +扇区 or +size{K,M,G} (6293504-20971519，默认为 20971519)：+2G
  分区 3 已设置为 Linux 类型，大小设为 2 GiB

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xab7f9d31

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     6293503     3145728    5  Extended
  /dev/sdb3         6293504    10487807     2097152   83  Linux

  命令(输入 m 获取帮助)：n	# 创建一个逻辑分区
  Partition type:
     p   primary (1 primary, 1 extended, 2 free)
     l   logical (numbered from 5)
  Select (default p): l
  添加逻辑分区 5
  起始 扇区 (4096-6293503，默认为 4096)：
  将使用默认值 4096
  Last 扇区, +扇区 or +size{K,M,G} (4096-6293503，默认为 6293503)：+1G
  分区 5 已设置为 Linux 类型，大小设为 1 GiB

  命令(输入 m 获取帮助)：p	# 可以看到逻辑分区的分区号从5开始

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0xab7f9d31

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdb1            2048     6293503     3145728    5  Extended
  /dev/sdb3         6293504    10487807     2097152   83  Linux
  /dev/sdb5            4096     2101247     1048576   83  Linux
  ```

## parted命令

- 由于mbr分区格式磁盘大小不能超过2T，所以GPT格式逐渐成为主流，GPT格式没有主分区、逻辑分区、扩展分区之分，在一块GPT格式的磁盘上，最多可以划分128个分区；
- 而`parted`则是支持对MBR和GPT格式磁盘进行分区的命令行工具，常用的功能如下：

<style>
table th:first-of-type {

    width: 100px;
}
table th {
    text-align: center;
}
</style>

| 常用功能命令     | 作用                                       |
| :----------: | ---------------------------------------- |
| `check`    | 检查文件系统，建议使用`fsck`命令检查                    |
| `mklabel`  | 创建分区表，即使用MBR还是GPT或其他分区表格式                |
| `mkfs`     | 创建文件系统，由于该命令不支持ext3，所以建议分区完成后使用其他命令创建文件系统 |
| `mkpart`   | 创建新分区，格式为`mkpart (part-type) (fs-type) start end`，`part-type`主要有`primary(主分区)`、`extended(扩展分区)`、`logical(逻辑分区)`，扩展和逻辑分区只对MBR格式有效，`fs-type`则是文件系统类型，如ext3、ext4，`start`&`end`则是分区起始和结束位置 |
| `mkpartfs` | 建立分区及其文件系统，与mkfs类似，不建议使用                 |
| `print`    | 输出分区信息，该命令有`free`显示该磁盘所以信息，并显示磁盘剩余空间；`number`显示指定分区信息；`all`显示所以磁盘信息三个选项 |
| `resize`   | 调整指定的分区大小                                |
| `rescue`   | 恢复被`parted`的`rm`命令删除的分区，需要给出分区的起始和结束位置，当查找到被删除的分区时，会提示恢复 |
| `rm`       | 删除分区，格式为`rm (分区编号)`                      |
| `select`   | 选择设备，存在多块磁盘时，需要选择操作的磁盘，格式为`select /dev/sdb` |
| `set`      | 更改指定分区编号的标志，标志有boot引导分区，hidden隐藏分区，raid软raid，lvm等，命令格式为`set 3 boot on`，即为设置3号分区为启动分区 |

## 使用parted对磁盘分区

- 使用`parted (磁盘分区路径)`对指定的磁盘进行操作，并使用`mklabel`命令将磁盘分区表改为GPT格式：

  ```bash
  [root@localhost ~]# parted /dev/sdb
  GNU Parted 3.1
  使用 /dev/sdb
  Welcome to GNU Parted! Type 'help' to view a list of commands.
  (parted) print                                                            
  Model: ATA VBOX HARDDISK (scsi)
  Disk /dev/sdb: 10.7GB
  Sector size (logical/physical): 512B/512B
  Partition Table: msdos
  Disk Flags: 

  Number  Start  End  Size  Type  File system  标志

  (parted) mklabel gpt    # 更改为GPT格式分区表                                                  
  警告: The existing disk label on /dev/sdb will be destroyed and all data on this disk will be lost. Do you want
  to continue?
  是/Yes/否/No? y                                                           
  (parted) print                                                            
  Model: ATA VBOX HARDDISK (scsi)
  Disk /dev/sdb: 10.7GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt	# 磁盘分区表格式已经修改成功
  Disk Flags: 

  Number  Start  End  Size  File system  Name  标志

  ```

- 新增分区使用`mkpart`命令，并指定为`primary`主分区：

  ```bash
  (parted) mkpart primary 1M 2G	#创建一个2G大小的主分区
  (parted) print                                                            
  Model: ATA VBOX HARDDISK (scsi)
  Disk /dev/sdb: 10.7GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags: 

  Number  Start   End     Size    File system  Name     标志
   1      1049kB  2000MB  1999MB               primary

  ```

- `parted`创建的分区不需要保存即可生效，使用`q`命令退出`parted`交互模式，再用`fdisk -l`查看已经新增的分区：

  ```bash
  [root@localhost ~]# fdisk -l                                              
  WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：gpt


  #         Start          End    Size  Type            Name
   1         2048      3905535    1.9G  Microsoft basic primary

  磁盘 /dev/sda：32.2 GB, 32212254720 字节，62914560 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0x00022876

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sda1   *        2048      411647      204800   83  Linux
  /dev/sda2          411648     4605951     2097152   82  Linux swap / Solaris
  /dev/sda3         4605952    62914559    29154304   83  Linux

  ```

- 将分区格式化为ext4格式，使用`mkfs.ext4`命令：

  ```bash
  [root@localhost ~]# mkfs.ext4 /dev/sdb1
  mke2fs 1.42.9 (28-Dec-2013)
  文件系统标签=
  OS type: Linux
  块大小=4096 (log=2)
  分块大小=4096 (log=2)
  Stride=0 blocks, Stripe width=0 blocks
  122160 inodes, 487936 blocks
  24396 blocks (5.00%) reserved for the super user
  第一个数据块=0
  Maximum filesystem blocks=501219328
  15 block groups
  32768 blocks per group, 32768 fragments per group
  8144 inodes per group
  Superblock backups stored on blocks: 
  	32768, 98304, 163840, 229376, 294912

  Allocating group tables: 完成                            
  正在写入inode表: 完成                            
  Creating journal (8192 blocks): 完成
  Writing superblocks and filesystem accounting information: 完成 

  [root@localhost ~]# parted /dev/sdb
  GNU Parted 3.1
  使用 /dev/sdb
  Welcome to GNU Parted! Type 'help' to view a list of commands.
  (parted) print                                                            
  Model: ATA VBOX HARDDISK (scsi)
  Disk /dev/sdb: 10.7GB
  Sector size (logical/physical): 512B/512B
  Partition Table: gpt
  Disk Flags: 

  Number  Start   End     Size    File system     Name     标志
   1      1049kB  2000MB  1999MB  ext4            primary
  ```


- `parted`也可以不进入交互模式而对磁盘进行分区操作，比如创建一个新的swap分区：

  ```bash
  [root@localhost ~]# parted /dev/sdb mkpart swap 2G 3G	# 创建一个1G大小的swap分区
  信息: You may need to update /etc/fstab.

  [root@localhost ~]# fdisk -l                                              
  WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：gpt


  #         Start          End    Size  Type            Name
   1         2048      3905535    1.9G  Microsoft basic primary
   2      3905536      5859327    954M  Microsoft basic swap

  ```

- 激活swap分区需要使用`mkswap`命令将分区格式化，然后使用`swapon`命令激活分区：

  ```bash
  [root@localhost ~]# mkswap /dev/sdb2
  正在设置交换空间版本 1，大小 = 976892 KiB
  无标签，UUID=596ad292-a0c4-4282-a1b6-340e4f88a83d
  [root@localhost ~]# swapon /dev/sdb2
  [root@localhost ~]# free
                total        used        free      shared  buff/cache   available
  Mem:        1016476       99792      750940        6720      165744      751612
  Swap:       3074040           0     3074040
  ```

- 由于`parted`命令是直接操作磁盘，所以当不小心删除分区时，使用`rescue`进行恢复：

  ```bash
  [root@localhost ~]# parted /dev/sdb rm 1	# 删除分区
  信息: You may need to update /etc/fstab.

  [root@localhost ~]# fdisk -l /dev/sdb
  WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：gpt


  #         Start          End    Size  Type            Name
   2      3905536      5859327    954M  Microsoft basic swap


  [root@localhost ~]# parted /dev/sdb rescue 1M 2G	# 恢复删除的分区
  正在搜索文件系统... 3%  (剩余时间 00:35)信息: A ext4 primary partition was found at 1049kB -> 2000MB.  Do you want to add it to the partition table?
  是/Yes/否/No/放弃/Cancel? yes                                             
  信息: You may need to update /etc/fstab.

  [root@localhost ~]# fdisk -l /dev/sdb                          
  WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：gpt


  #         Start          End    Size  Type            Name
   1         2048      3905535    1.9G  Microsoft basic 
   2      3905536      5859327    954M  Microsoft basic swap

  ```

---

