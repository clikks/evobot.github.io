---
title: Linux磁盘管理—分区格式化与挂载
author: Evobot
categories: Centos7
tags:
  - Centos
abbrlink: aa04cc87
date: 2018-04-10 23:33:44
image:
---



![磁盘分区](https://s1.ax1x.com/2020/04/28/JIdjpT.png)

本文主要介绍如何格式化新增的磁盘分区，并介绍怎样将新分区挂载到系统目录，以及如何增加swap空间。

<!-- more -->

---

# 磁盘格式化

- 磁盘分区之后必须进行格式化才能够使用，Centos7支持的文件系统可以通过命令`cat /etc/filesystems`进行查看：

  ```bash
  [lux@evobot ~]$ cat /etc/filesystems 
  xfs
  ext4
  ext3
  ext2
  nodev proc
  nodev devpts
  iso9660
  vfat
  hfs
  hfsplus
  *
  ```

- Centos7默认的文件系统为`xfs`，查看已挂载的分区的文件系统，可以使用`mount`命令查看，执行命令后结果输出比较多，只需要关注`/dev/sdx`这样的分区即可，如果查看未挂载的分区文件系统，使用`blkid`命令：

  ```bash
  [root@localhost ~]# mount | column -t	# column -t可以使输出内容比较整齐
  sysfs       on  /sys                             type  sysfs       (rw,nosuid,nodev,noexec,relatime,seclabel)
  proc        on  /proc                            type  proc        (rw,nosuid,nodev,noexec,relatime)
  devtmpfs    on  /dev                             type  devtmpfs    (rw,nosuid,seclabel,size=498656k,nr_inodes=124664,mode=755)
  securityfs  on  /sys/kernel/security             type  securityfs  (rw,nosuid,nodev,noexec,relatime)
  tmpfs       on  /dev/shm                         type  tmpfs       (rw,nosuid,nodev,seclabel)
  devpts      on  /dev/pts                         type  devpts      (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
  tmpfs       on  /run                             type  tmpfs       (rw,nosuid,nodev,seclabel,mode=755)
  /dev/sda3   on  /                                type  xfs         (rw,relatime,seclabel,attr2,inode64,noquota)
  selinuxfs   on  /sys/fs/selinux                  type  selinuxfs   (rw,relatime)
  /dev/sda1   on  /boot                            type  xfs         (rw,relatime,seclabel,attr2,inode64,noquota)

  [root@localhost ~]# blkid /dev/sdb1
  /dev/sdb1: UUID="708dbd6b-e75f-4196-98b0-0457525b8e97" TYPE="ext4" PARTUUID="fa0fcddc-d811-44e8-bfe0-d8f38384bbc5" 

  ```

## mke2fs命令

- 格式化分区使用`mke2fs`命令，此命令不支持`xfs`文件系统的格式化，常用选项如下：
<style>
table th:first-of-type {

    width: 70px;
}
table th {
    text-align: center;
}
</style>

| 命令选项 | 作用                                       |
| :----: | ---------------------------------------- |
| `-t` | 指定文件系统类型，如ext3、ext4，不指定时默认为ext2          |
| `-b` | 指定分区块大小，默认为4KB，支持1024B、2048B等，使用`du -sb`可以查看文件实际大小 |
| `-m` | 指定分区预留空间大小，默认预留5%，可以指定为1、0.1等            |
| `-i` | 指定多少字节对应inode大小，默认4个块对应一个inode，即16K对应一个inode，可以指定为8192，即2个块对应一个inode |

```bash
[root@localhost ~]# mke2fs -t ext3 -b 2048 -m 1 /dev/sdb1
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=
OS type: Linux
块大小=2048 (log=1)
分块大小=2048 (log=1)
Stride=0 blocks, Stripe width=0 blocks
122400 inodes, 975872 blocks
9758 blocks (1.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=537919488
60 block groups
16384 blocks per group, 16384 fragments per group
2040 inodes per group
Superblock backups stored on blocks: 
	16384, 49152, 81920, 114688, 147456, 409600, 442368, 802816

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (16384 blocks): 完成
Writing superblocks and filesystem accounting information: 完成 

[root@localhost ~]# mke2fs -t ext3  -m 1 -i 8192 /dev/sdb1
mke2fs 1.42.9 (28-Dec-2013)
文件系统标签=
OS type: Linux
块大小=4096 (log=2)
分块大小=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
244080 inodes, 487936 blocks
4879 blocks (1.00%) reserved for the super user
第一个数据块=0
Maximum filesystem blocks=503316480
15 block groups
32768 blocks per group, 32768 fragments per group
16272 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912

Allocating group tables: 完成                            
正在写入inode表: 完成                            
Creating journal (8192 blocks): 完成
Writing superblocks and filesystem accounting information: 完成
```

## mkfs.xfs命令

- 格式化为`xfs`文件系统，使用`mkfs.xfs`命令，当分区已经存在文件系统时，需要加`-f`选项：

  ```bash
  [root@localhost ~]# mkfs.xfs /dev/sdb1
  mkfs.xfs: /dev/sdb1 appears to contain an existing filesystem (ext3).
  mkfs.xfs: Use the -f option to force overwrite.

  [root@localhost ~]# mkfs.xfs -f /dev/sdb1
  meta-data=/dev/sdb1              isize=512    agcount=4, agsize=121984 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=0, sparse=0
  data     =                       bsize=4096   blocks=487936, imaxpct=25
           =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
  log      =internal log           bsize=4096   blocks=2560, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0

  ```

- `mkfs.xfs`同样也可以指定文件系统的块大小，使用`-b size=(num)`选项：

  ```bash
  [root@localhost ~]# mkfs.xfs -b size=2048 -f /dev/sdb1
  meta-data=/dev/sdb1              isize=512    agcount=4, agsize=243968 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=0, sparse=0
  data     =                       bsize=2048   blocks=975872, imaxpct=25
           =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
  log      =internal log           bsize=2048   blocks=5120, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0

  ```

---

# 磁盘挂载
## mount命令
- `mount`命令用来对分区进行挂载，命令用法为`mount [-o] /dev/sdx`,`-o`参数可以指定挂载的选项，如`rw`,`exec`.`async`等
```bash
[root@localhost ~]# mount /dev/sdb /mnt/
[root@localhost ~]# df -h
文件系统        容量  已用  可用 已用% 挂载点
/dev/sda3        48G  1.3G   47G    3% /
devtmpfs        479M     0  479M    0% /dev
tmpfs           489M     0  489M    0% /dev/shm
tmpfs           489M  6.7M  482M    2% /run
tmpfs           489M     0  489M    0% /sys/fs/cgroup
/dev/sda1       197M  109M   88M   56% /boot
tmpfs            98M     0   98M    0% /run/user/0
#/dev/sdb就是我们所挂载的磁盘
/dev/sdb         10G   33M   10G    1% /mnt
```
## umount命令
- `umount`命令用来卸载已挂载的磁盘，命令用法为`umount [-l] [disk|dir]`，可以使用`/dev/`下的磁盘路径卸载，也可以使用磁盘挂载的目录卸载：
```bash
[root@localhost mnt]# umount /dev/sdb 
umount: /mnt：目标忙。
        (有些情况下通过 lsof(8) 或 fuser(1) 可以
         找到有关使用该设备的进程的有用信息)
[root@localhost mnt]# umount -l /mnt/
[root@localhost mnt]# df -h
文件系统        容量  已用  可用 已用% 挂载点
/dev/sda3        48G  1.3G   47G    3% /
devtmpfs        479M     0  479M    0% /dev
tmpfs           489M     0  489M    0% /dev/shm
tmpfs           489M  6.7M  482M    2% /run
tmpfs           489M     0  489M    0% /sys/fs/cgroup
/dev/sda1       197M  109M   88M   56% /boot
tmpfs            98M     0   98M    0% /run/user/0
```
## fstab文件
- `/etc/fstab`文件是用来开机挂载磁盘的文件，文件内容分为5列，分别是UUID,挂载点，挂载选项，dump备份，开机启动优先级，其中dump和优先级默认为0，若开启优先级，则根分区为1，其他分区为2：
```bash
[root@localhost mnt]# cat /etc/fstab 
#
# /etc/fstab
# Created by anaconda on Sat Aug 19 07:18:38 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=e54aee54-8247-4abd-a51b-ae2d89416133 /                       xfs     defaults        0 0
UUID=8916b3db-57d9-4c03-831e-542d7550b1a2 /boot                   xfs     defaults        0 0
UUID=9670b2c4-e1e1-4214-9b4e-7e840a36deb2 swap                    swap    defaults        0 0
```
- 查看磁盘的UUID,使用`blkid`命令查看,除了使用UUID,也可以使用磁盘路径`/dev/sdx`写入`fstab`文件.
```bash
[root@localhost mnt]# cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Sat Aug 19 07:18:38 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=e54aee54-8247-4abd-a51b-ae2d89416133 /                       xfs     defaults        0 0
UUID=8916b3db-57d9-4c03-831e-542d7550b1a2 /boot                   xfs     defaults        0 0
UUID=9670b2c4-e1e1-4214-9b4e-7e840a36deb2 swap                    swap    defaults        0 0
#下面就是我们自己添加的需要挂载的磁盘
UUID=5b0de14c-5a26-43ec-9d13-fa4440072c82    /mnt        xfs    defaults    0 0
```
- 写入`fstab`文件之后,使用命令`mount -a`来检查`fstab`文件的挂载内容是否有错误,如果没有问题,则磁盘会被顺利挂载。

------------

# 创建swap分区
- `swap`分区是系统的虚拟内存,一般来说为内存的2倍,但是一般最大8G即可,当系统需要较大的swap空间时,可以手动增加swap分区。
## 创建虚拟磁盘
- 创建虚拟磁盘使用`dd`命令,命令格式为`dd if=/dev/zero of=/tmp/tmpimage.img bs=1M count=100`
```bash
[root@localhost mnt]# dd if=/dev/zero of=/tmp/virtual.img bs=1M count=100
记录了100+0 的读入
记录了100+0 的写出
104857600字节(105 MB)已复制，2.72497 秒，38.5 MB/秒
```
## 设置swap分区
- 格式化`swap`分区使用`mkswap`命令:
```bash
[root@localhost mnt]# mkswap -f /tmp/virtual.img 
正在设置交换空间版本 1，大小 = 102396 KiB
无标签，UUID=cf9a3f57-1ddc-4e5a-9766-bb11aa7d526b
```
- 使用`swapon`命令将新的`swap`挂载到系统:
```bash
[root@localhost mnt]# free -h
              total        used        free      shared  buff/cache   available
Mem:           976M        109M        608M        6.6M        258M        690M
Swap:          2.0G          0B        2.0G
[root@localhost mnt]# swapon /tmp/virtual.img 
swapon: /tmp/virtual.img：不安全的权限 0644，建议使用 0600。
[root@localhost mnt]# free -h    #查看swap空间已经增加
              total        used        free      shared  buff/cache   available
Mem:           976M        109M        608M        6.6M        258M        690M
Swap:          2.1G          0B        2.1G
```
- 卸载swap分区,使用`swapoff`命令:
```bash
[root@localhost mnt]# swapoff /tmp/virtual.img 
[root@localhost mnt]# free -h
              total        used        free      shared  buff/cache   available
Mem:           976M        109M        608M        6.6M        258M        690M
Swap:          2.0G          0B        2.0G
```
---

