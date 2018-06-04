---
title: Linux磁盘管理——LVM磁盘分区
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 8901b9bb
date: 2018-04-11 22:12:46
image:
---



LVM是指逻辑卷管理，由物理磁盘、物理卷、卷组、逻辑卷组成。物理卷组在物理磁盘上创建，磁盘ID为`8e`，与分区创建相同，一个和多个物理卷组成卷组，在卷组的基础上划分逻辑卷，逻辑卷格式化之后进行挂载。

![LVM](http://p5qynomrl.bkt.clouddn.com/1523456458706v7d50vlz.png?imageslim)

LVM相比普通分区，优点在于可以方便的进行扩容和缩容，但是同样也带来了数据恢复困难的缺点。

<!--more-->

---

# LVM物理卷

## 创建物理分区

- 创建物理分区与磁盘分区相同，但需要在`fdisk`中将分区ID设置为`8e`，`parted`中则使用`set`设置为`lvm`，然后使用`pvcreate`命令创建物理卷。

### 使用fdisk分区

- 使用`fdisk`分区，与之前的磁盘分区一样，只需要使用`t`将分区ID更改为`8e`即可：

  ```bash
  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdc：4294 MB, 4294967296 字节，8388608 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0x3f2e9a4a

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdc1            2048     4196351     2097152   83  Linux

  命令(输入 m 获取帮助)：t	# 将分区ID更改为LVM
  已选择分区 1
  Hex 代码(输入 L 列出所有代码)：8e	# 可以输入L列出所以的分区类型代码
  已将分区“Linux”的类型更改为“Linux LVM”

  命令(输入 m 获取帮助)：p

  磁盘 /dev/sdc：4294 MB, 4294967296 字节，8388608 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：dos
  磁盘标识符：0x3f2e9a4a

     设备 Boot      Start         End      Blocks   Id  System
  /dev/sdc1            2048     4196351     2097152   8e  Linux LVM
  ```

### 使用parted分区

- 使用`parted`对GPT格式的磁盘进行分区，同样也需要将分区类型设置为LVM，使用`parted`的`set`命令即可：

  ```bash
  [root@localhost ~]# parted /dev/sdb mkpart primary 1M 2G set 1 lvm
  新状态？  [开]/on/关/off? on                                              
  信息: You may need to update /etc/fstab.

  [root@localhost ~]# fdisk -l /dev/sdb
  WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

  磁盘 /dev/sdb：10.7 GB, 10737418240 字节，20971520 个扇区
  Units = 扇区 of 1 * 512 = 512 bytes
  扇区大小(逻辑/物理)：512 字节 / 512 字节
  I/O 大小(最小/最佳)：512 字节 / 512 字节
  磁盘标签类型：gpt


  #         Start          End    Size  Type            Name
   1         2048      3905535    1.9G  Linux LVM       primary

  ```

## 创建物理卷

- 创建物理卷使用`pvcreate`命令，这个命令在`lvm`软件包内，默认并没有安装，使用`yum install -y lvm2`安装；

> 当需要使用一个命令，但命令所在的软件包没有安装，并且不知道软件包的名字的时候，可以使用`yum provides "/*/pvcreate"`命令查找命令所在的软件包，其中`"/*/pvcreate"`是命令的路径，这里使用通配的方式查找。

- 使用命令`pvcreate /dev/sdXX`将分区创建为LVM物理卷：

  ```bash
  [root@localhost ~]# pvcreate /dev/sdb1
  WARNING: ext3 signature detected on /dev/sdb1 at offset 1080. Wipe it? [y/n]: y	# 由于分区存在文件系统，需要清除
    Wiping ext3 signature on /dev/sdb1.
    Physical volume "/dev/sdb1" successfully created.
  ```

> 有时候创建完分区后，在`/dev/`目录下新的分区`sdb1`可能并没有自动生成文件，这时可以使用命令`partprobe`来生成。

- 使用`pvdisplay`命令查看物理卷，命令执行结果会列出物理卷的详细信息：

  ```bash
  [root@localhost ~]# pvdisplay 
    "/dev/sdb2" is a new physical volume of "1.86 GiB"
    --- NEW Physical volume ---
    PV Name               /dev/sdb2
    VG Name               
    PV Size               1.86 GiB
    Allocatable           NO
    PE Size               0   
    Total PE              0
    Free PE               0
    Allocated PE          0
    PV UUID               GxoHRk-qwFY-xIPx-fnSK-Oj9w-9A9V-3cIrei
     
    "/dev/sdb1" is a new physical volume of "1.86 GiB"
    --- NEW Physical volume ---
    PV Name               /dev/sdb1
    VG Name               
    PV Size               1.86 GiB
    Allocatable           NO
    PE Size               0   
    Total PE              0
    Free PE               0
    Allocated PE          0
    PV UUID               qGXHcS-DYtM-rV90-QChT-Sy0L-qKff-qbfWm3

  ```

- 使用`pvs`命令可以查看物理卷的简要信息：

  ```bash
  [root@localhost ~]# pvs
    PV         VG Fmt  Attr PSize  PFree 
    /dev/sdb1     lvm2 ---   1.86g  1.86g
    /dev/sdb2     lvm2 ---   1.86g  1.86g
    /dev/sdc1     lvm2 ---   2.00g  2.00g
    /dev/sdc2     lvm2 ---  <2.00g <2.00g

  ```

---

# LVM卷组

## 创建卷组

- 创建LVM卷组，使用`vgcreate`命令，命令格式为`vgcreate (卷组名) (分区名)`，其中分区名可以是多个：

  ```bash
  [root@localhost ~]# vgcreate vg1 /dev/sdb1 /dev/sdb2
    Volume group "vg1" successfully created

  ```

- 查看已创建的卷组，使用`vgdisplay`或`vgs`命令：

  ```bash
  [root@localhost ~]# vgdisplay 
    --- Volume group ---
    VG Name               vg1
    System ID             
    Format                lvm2
    Metadata Areas        2
    Metadata Sequence No  1
    VG Access             read/write
    VG Status             resizable
    MAX LV                0
    Cur LV                0
    Open LV               0
    Max PV                0
    Cur PV                2
    Act PV                2
    VG Size               <3.72 GiB
    PE Size               4.00 MiB
    Total PE              952
    Alloc PE / Size       0 / 0   
    Free  PE / Size       952 / <3.72 GiB
    VG UUID               EBbYQ5-v6Gh-U2d4-ZEVS-KO21-FHYk-qaWPib

  [root@localhost ~]# vgs
    VG  #PV #LV #SN Attr   VSize  VFree 
    vg1   2   0   0 wz--n- <3.72g <3.72g

  ```

- 使用命令`vgremove`可以删除卷组：

  ```bash
  [root@localhost ~]# vgremove vg1
    Volume group "vg1" successfully removed

  ```

## 卷组扩容

- 卷组扩容使用`vgextend`命令，命令用法为`vgextend (卷组) (物理卷)`:

  ```bash
  [root@localhost ~]# pvs
    PV         VG  Fmt  Attr PSize  PFree  
    /dev/sdb1  vg1 lvm2 a--  <1.86g 880.00m
    /dev/sdb2  vg1 lvm2 a--  <1.86g  <1.86g
    /dev/sdc1      lvm2 ---   2.00g   2.00g
    /dev/sdc2      lvm2 ---  <2.00g  <2.00g
    
  [root@localhost ~]# vgextend vg1 /dev/sdc1
    Volume group "vg1" successfully extended
    
  [root@localhost ~]# vgs
    VG  #PV #LV #SN Attr   VSize VFree
    vg1   3   1   0 wz--n- 5.71g 4.71g
   
  [root@localhost ~]# pvs
    PV         VG  Fmt  Attr PSize  PFree  
    /dev/sdb1  vg1 lvm2 a--  <1.86g 880.00m
    /dev/sdb2  vg1 lvm2 a--  <1.86g  <1.86g
    /dev/sdc1  vg1 lvm2 a--  <2.00g  <2.00g
    /dev/sdc2      lvm2 ---  <2.00g  <2.00g

  ```

  ​

---

# LVM逻辑卷

## 创建逻辑卷

- 创建逻辑卷使用命令`lvcreate`，命令用法为`lvcreate -L (逻辑卷大小) -n (逻辑卷名) (卷组名)`：

  ```bash
  [root@localhost ~]# lvcreate -L 500M -n lv1 vg1
    Logical volume "lv1" created.

  ```

- 查看已创建的逻辑卷，使用`lvdisplay`或`lvs`命令：

  ```bash
  [root@localhost ~]# lvdisplay 
    --- Logical volume ---
    LV Path                /dev/vg1/lv1
    LV Name                lv1
    VG Name                vg1
    LV UUID                y325TB-keCq-Gl9C-Oh1V-7rMp-CCHe-LjHHfx
    LV Write Access        read/write
    LV Creation host, time localhost.localdomain, 2018-04-11 23:29:28 +0800
    LV Status              available
    # open                 0
    LV Size                500.00 MiB
    Current LE             125
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     8192
    Block device           253:0

  [root@localhost ~]# lvs
    LV   VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    lv1  vg1 -wi-a----- 500.00m                           
  ```

## 逻辑卷格式化

- 逻辑卷的格式化与分区的格式化相同，只是LVM逻辑卷文件路径与分区文件路径不同：

  ```bash
  [root@localhost ~]# mkfs.ext4 /dev/vg1/lv1 	# 格式化为ext4
  mke2fs 1.42.9 (28-Dec-2013)
  文件系统标签=
  OS type: Linux
  块大小=1024 (log=0)
  分块大小=1024 (log=0)
  Stride=0 blocks, Stripe width=0 blocks
  128016 inodes, 512000 blocks
  25600 blocks (5.00%) reserved for the super user
  第一个数据块=1
  Maximum filesystem blocks=34078720
  63 block groups
  8192 blocks per group, 8192 fragments per group
  2032 inodes per group
  Superblock backups stored on blocks: 
  	8193, 24577, 40961, 57345, 73729, 204801, 221185, 401409

  Allocating group tables: 完成                            
  正在写入inode表: 完成                            
  Creating journal (8192 blocks): 完成
  Writing superblocks and filesystem accounting information: 完成 

  [root@localhost ~]# mkfs.xfs -f /dev/vg1/lv1 	#格式化为xfs
  meta-data=/dev/vg1/lv1           isize=512    agcount=4, agsize=32000 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=0, sparse=0
  data     =                       bsize=4096   blocks=128000, imaxpct=25
           =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
  log      =internal log           bsize=4096   blocks=855, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0

  ```

## 逻辑卷挂载

- 挂在逻辑卷与分区挂载相同，使用`mount`命令：

  ```bash
  [root@localhost ~]# mount /dev/vg1/lv1 /mnt/
  [root@localhost ~]# df -h
  文件系统             容量  已用  可用 已用% 挂载点
  /dev/sda3             28G  1.2G   27G    5% /
  devtmpfs             487M     0  487M    0% /dev
  tmpfs                497M     0  497M    0% /dev/shm
  tmpfs                497M  6.6M  490M    2% /run
  tmpfs                497M     0  497M    0% /sys/fs/cgroup
  /dev/sda1            197M  108M   89M   55% /boot
  tmpfs                100M     0  100M    0% /run/user/0
  /dev/mapper/vg1-lv1  497M   26M  472M    6% /mnt

  ```

- 使用`df -h`查看挂载发现逻辑卷的文件路径与`mount`时指定的文件不同，实际上两个路径都是指向同一个文件：

  ```bash
  [root@localhost ~]# ls -l /dev/vg1/lv1 
  lrwxrwxrwx. 1 root root 7 4月  11 23:33 /dev/vg1/lv1 -> ../dm-0
  [root@localhost ~]# ls -l /dev/mapper/vg1-lv1 
  lrwxrwxrwx. 1 root root 7 4月  11 23:33 /dev/mapper/vg1-lv1 -> ../dm-0

  ```

## 逻辑卷扩缩容

### ext文件系统扩容

- 扩容逻辑卷使用命令`lvresize`，用法为`lvresize -L (逻辑卷大小) (逻辑卷路径)`，扩容前需要现将逻辑卷取消挂载：

  ```bash
  [root@localhost ~]# umount /dev/vg1/lv1 
  [root@localhost ~]# lvresize -L 800M /dev/vg1/lv1 
    Size of logical volume vg1/lv1 changed from 500.00 MiB (125 extents) to 800.00 MiB (200 extents).
    Logical volume vg1/lv1 successfully resized.

  ```

- 扩容后需要检查逻辑卷是否存在错误，使用`e2fsck -f (逻辑卷)`命令进行检查：

  ```bash
  [root@localhost ~]# lvresize -L 800M /dev/vg1/lv1 
    Size of logical volume vg1/lv1 changed from 500.00 MiB (125 extents) to 800.00 MiB (200 extents).
    Logical volume vg1/lv1 successfully resized.
  [root@localhost ~]# e2fsck -f /dev/vg1/lv1 
  e2fsck 1.42.9 (28-Dec-2013)
  第一步: 检查inode,块,和大小
  第二步: 检查目录结构
  第3步: 检查目录连接性
  Pass 4: Checking reference counts
  第5步: 检查簇概要信息
  /dev/vg1/lv1: 12/128016 files (0.0% non-contiguous), 26686/512000 blocks

  ```

- 然后需要更新逻辑卷的信息，使用命令`resize2fs (逻辑卷)`进行更新：

  ```bash
  root@localhost ~]# resize2fs /dev/vg1/lv1 
  resize2fs 1.42.9 (28-Dec-2013)
  Resizing the filesystem on /dev/vg1/lv1 to 819200 (1k) blocks.
  The filesystem on /dev/vg1/lv1 is now 819200 blocks long.
  ```

- 然后重新挂载逻辑卷即可：

  ```bash
  [root@localhost ~]# !mount
  mount /dev/vg1/lv1 /mnt/
  [root@localhost ~]# cat /mnt/test.txt 
  1123456
  ```

### ext文件系统缩容

- xfs文件系统不支持缩容，缩容操作需要先取消逻辑卷的挂载，然后使用`e2fsck`命令检查磁盘错误，之后更新逻辑卷的信息，最后再设置逻辑卷大小，步骤基本与扩容相反，但是在更新逻辑卷信息时，需要指定新的逻辑卷大小，使用`resize2fs (逻辑卷) (缩容后逻辑卷大小)`：

  ```bash
  [root@localhost ~]# !umount
  umount /dev/vg1/lv1 

  [root@localhost ~]# e2fsck -f /dev/vg1/lv1 	#检查磁盘错误
  e2fsck 1.42.9 (28-Dec-2013)
  第一步: 检查inode,块,和大小
  第二步: 检查目录结构
  第3步: 检查目录连接性
  Pass 4: Checking reference counts
  第5步: 检查簇概要信息
  /dev/vg1/lv1: 12/203200 files (0.0% non-contiguous), 36419/819200 blocks

  [root@localhost ~]# resize2fs /dev/vg1/lv1 500M	# 更新逻辑卷信息
  resize2fs 1.42.9 (28-Dec-2013)
  Resizing the filesystem on /dev/vg1/lv1 to 512000 (1k) blocks.
  The filesystem on /dev/vg1/lv1 is now 512000 blocks long.

  [root@localhost ~]# lvresize -L 500M /dev/vg1/lv1 #设置新的大小
    WARNING: Reducing active logical volume to 500.00 MiB.
    THIS MAY DESTROY YOUR DATA (filesystem etc.)
  Do you really want to reduce vg1/lv1? [y/n]: y
    Size of logical volume vg1/lv1 changed from 800.00 MiB (200 extents) to 500.00 MiB (125 extents).
    Logical volume vg1/lv1 successfully resized.

  [root@localhost ~]# lvdisplay 
    --- Logical volume ---
    LV Path                /dev/vg1/lv1
    LV Name                lv1
    VG Name                vg1
    LV UUID                dKQ2IM-hFhJ-iXN9-f5WK-HPrI-dB1a-0P7Q5i
    LV Write Access        read/write
    LV Creation host, time localhost.localdomain, 2018-04-11 23:51:47 +0800
    LV Status              available
    # open                 0
    LV Size                500.00 MiB
    Current LE             125
    Segments               1
    Allocation             inherit
    Read ahead sectors     auto
    - currently set to     8192
    Block device           253:0
     
  [root@localhost ~]# lvs
    LV   VG  Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    lv1  vg1 -wi-a----- 500.00m                                                    
  ```

### xfs文件系统扩容

- `xfs`文件系统的逻辑卷在扩容时不需要卸载，也不需要检查磁盘错误和更新磁盘信息，只需要直接使用`lvresize`命令扩容即可：

  ```bash
  [root@localhost ~]# lvresize -L 1G /dev/vg1/lv1 
    Size of logical volume vg1/lv1 changed from 500.00 MiB (125 extents) to 1.00 GiB (256 extents).
    Logical volume vg1/lv1 successfully resized.
    
  [root@localhost ~]# lvs
    LV   VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
    lv1  vg1 -wi-ao---- 1.00g                                                    
  [root@localhost ~]# df -h
  文件系统             容量  已用  可用 已用% 挂载点
  /dev/sda3             28G  1.2G   27G    5% /
  devtmpfs             487M     0  487M    0% /dev
  tmpfs                497M     0  497M    0% /dev/shm
  tmpfs                497M  6.6M  490M    2% /run
  tmpfs                497M     0  497M    0% /sys/fs/cgroup
  /dev/sda1            197M  108M   89M   55% /boot
  tmpfs                100M     0  100M    0% /run/user/0
  /dev/mapper/vg1-lv1  497M   26M  472M    6% /mnt

  ```

- 扩容后通过`df -h`查看挂载信息，逻辑卷的大小并没有更新，需要使用`xfs_growfs`命令进行更新，用法为`xfs_growfs (逻辑卷)`，执行这个命令时，磁盘必须已经挂载：

  ```bash
  [root@localhost ~]# xfs_growfs /dev/vg1/lv1 
  meta-data=/dev/mapper/vg1-lv1    isize=512    agcount=4, agsize=32000 blks
           =                       sectsz=512   attr=2, projid32bit=1
           =                       crc=1        finobt=0 spinodes=0
  data     =                       bsize=4096   blocks=128000, imaxpct=25
           =                       sunit=0      swidth=0 blks
  naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
  log      =internal               bsize=4096   blocks=855, version=2
           =                       sectsz=512   sunit=0 blks, lazy-count=1
  realtime =none                   extsz=4096   blocks=0, rtextents=0
  data blocks changed from 128000 to 262144

  [root@localhost ~]# df -h
  文件系统             容量  已用  可用 已用% 挂载点
  /dev/sda3             28G  1.2G   27G    5% /
  devtmpfs             487M     0  487M    0% /dev
  tmpfs                497M     0  497M    0% /dev/shm
  tmpfs                497M  6.6M  490M    2% /run
  tmpfs                497M     0  497M    0% /sys/fs/cgroup
  /dev/sda1            197M  108M   89M   55% /boot
  tmpfs                100M     0  100M    0% /run/user/0
  /dev/mapper/vg1-lv1 1021M   26M  996M    3% /mnt

  ```

> 当错误配置`fstab`文件重启之后，系统启动会进入故障模式，这时候需要输入root密码进入系统，然后正确修改`fstab`文件后重启即可恢复。

---

