---
title: windows10 & Arch 双系统安装指南【一】
tags:
  - linux
  - archlinux
  - cinnamon
  - windows10
categories: Linux
abbrlink: 3fd6800f
date: 2016-02-14 16:02:53
---
![](http://7xqo9u.com1.z0.glb.clouddn.com/arch%20cinnamonarch_linux_flat_wallpaper_by_debasishpatra-d7gmn9w.jpg)

Arch Linux是一款简洁优雅的GNU/Linux发行版，采用滚动升级的形式，只提供最基础的系统，一切配置都需要用户根据需要自行解决。

   Arch 的哲学：Keep It Simple, Stupid

Cinnamon是一款很适合上手的Linux桌面，桌面布局类似GNOME 2；然而，底层的技术又是基于GNOME Shell（GNOME 3）。目前Cinnamon是最流行的发行版LinuxMint的默认桌面环境。
<!-- more -->

---
# 安装准备
## 系统镜像
　　到arch的官网可以下载到最新的[系统镜像](https://www.archlinux.org/download/)，然后使用[刻录软件](http://pan.baidu.com/s/1o6WG5yM)将镜像刻录到U盘中。
## 引导安装
　　我是使用的UEFI模式引导的安装镜像，并且需要创建Windows10和Arch双系统。引导启动后会进入到arch的命令行模式。
## 网络配置
### 有线网络
　　进入系统后首先需要配置好网络才能继续后面的安装，arch的系统安装高度依赖网络，安装程序会在启动时自动运行**dhcpcd**守护进程以尝试有线连接。可以使用和**ping**命令检查网络是否正常，如果网络不通，可以参照[arch wiki](https://wiki.archlinux.org/index.php/Beginners%27_guide)解决。
### 无线网络
　　使用**iw dev**命令查看无线网卡的Interface名称，一般Interface名称为**wlp**开头，然后使用**wifi-menu [Interface]**连接wifi。
## 磁盘分区
　　因为我们采用UEFI启动并且硬盘已经安装了windows10系统，所以默认磁盘已经是GPT分区格式。执行命令：
```bash
fdisk -l
```
　　可以看到现在的硬盘分区情况，**/dev/sda**这就是我们的硬盘名称，如果有多块硬盘，则命名就是sdb、sdc等。
　　安装windows10之后，在sda上会有四个分区，其中有个100M的sda2是EFI分区，也就是启动分区，这4个分区我们都不需要重新分区，我们只需要创建arch所需要的分区就可以了。
　　我的分区方案是 **/** 分区20G，**swap** 分区4G，其余的分给 **home** 分区，我的windows分区中最后一个分区sda4的结束位置是41.9G,所以arch的分区就要从这里开始：
```bash
parted /dev/sda
mkpart primary ext4 41.9G 62G
mkpart primary linux-swap 62G 66G
mkpart primart ext4 66G -1
```
### 格式化分区
　　分区完成后退出parted分区，将分区格式化为指定的文件系统。
```bash
#查看所有分区，前4个分区都是windows的分区，我们格式化只针对后面几个分区。
lsblk /dev/sda   
```
```bash
mkfs.ext4 /dev/sda5
mkfs.ext4 /dev/sda7
mkswap /dev/sda6 #格式化swap分区
swapon /dev/sda6 #激活swap分区
```
### 挂载分区
　　先挂载 **/** (root)分区，其他目录都要在 **/** 分区中创建，然后再挂载。在安装环境中，使用 **/mnt** 目录挂载 **root** 分区：
```bash
mount /dev/sda5 /mnt  #挂载根分区
mkdir -p /mnt/{home,boot}  #创建home目录
mount /dev/sda7 /mnt/home  #挂载home分区到home目录
mount /dev/sda2 /mnt/boot #将windows的EFI分区挂载到boot目录
```
# 安装
## 配置安装镜像
　　如果想要安装某个软件，必须先从 **/etc/pacman.d/mirrorlist** 中定义的镜像站中下载安装包到本地。在live系统里，该文件中所有的镜像站都默认开启，并且按照镜像系统被创建时各镜像站的与官方镜像站的同步状态和速度来排序。
　　当系统在下载软件包的时候，列表中排的越靠前的镜像站优先级越高。所以我们需要将国内的镜像源放在前面，提高下载速度：
```bash
vi /etc/pacman.d/mirrorlist
#将下面两个镜像源复制到最前面
Server = http://mirrors.ustc.edu.cn/archlinux/$repo/os/$arch
Server = http://mirrors.163.com/archlinux/$repo/os/$arch
```
## 安装基础软件包
　　现在可以使用**pacstrap**安装基本系统：
```bash
pacstrap -i /mnt base base-devel
```
# 系统配置
## 生成分区表
```bash
genfstab -U -p /mnt > /mnt > /mnt/etc/fstab
```
## chroot
```bash
arch-chroot /mnt /bin/bash
```
## 本地化配置
```bash
vi /etc/locale.gen
#找到下面几项并去掉前面的＃号
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
```
```bash
#执行locale-gen生成locale信息
locale-gen
```
```bash
#创建locale.conf并提交本地化选项
echo LANG=en_US.UTF-8 > /etc/locale.conf
```
## 时间配置
```bash
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc --utc
```
## 设置Root密码
```bash
passwd  #输入两次密码
```
## 设置主机名
```bash
echo Arch > /etc/hostname
#修改主机名与上面相同
vi /etc/hosts 

#<ip-address>	<hostname.domain.org>	<hostname>
127.0.0.1			localhost.localdomain  		arch
::1				localhost.localdomain  		arch
```
## 安装引导程序
　　因为我们使用的是UEFI+GPT分区表的形式，所以我们安装**systemd-boot**到EFI分区：
```bash
#安装systemd-boot
bootctl install
#使用示例配置文件创建引导入口
cp /usr/share/systemd/bootctl/arch.conf /boot/loader/entries/
#查看根分区的PARTUUID
blkid -s PARTUUID -o value /dev/sda5
```
```bash
#修改引导入口
vi /boot/loader/entries/arch,conf
#将文件中PARTUUID后的XXX修改为刚刚获取的根目录的PARTUUID，rootfstype=XXX修改为ext4
  title   Arch Linux
  linux   /vmlinuz-linux
  initrd  /initramfs-linux.img
  options root=PARTUUID=2c3931f1-9659-4ad0-aa50-0b54754472bb rootfstype=ext4 add_efi_memmap
```
```bash
#修改启动等待时间，去掉timeout前面的#号
vi /boot/loader/loader.conf
  timeout 3
```
不需要配置windows的启动项，系统会自动读取windows的启动项并在开机时列出。
## 网络配置
　　前面已经配置过网络，这次我们是为新系统配置的网络，使得重启之后的新系统能够正常使用网络。
### 无线网络
安装**iw,dialog**和**wpa_supplicant**
```bash
pacman -S iw wpa_supplicant dialog
```
### 有线网络
有线网络只需要开机启动**dhcpcd**服务即可：
```bash
systemctl enable dhcpcd.service
```
## 卸载分区并重启
离开chroot环境然后就可以重启系统，进入我们的新系统了：
```bash
exit
reboot
```
　　重启时可以看到我们启动选项有archlinux和windows，选择archlinux，就可以进入我们的新系统了，当然现在系统只有命令行模式，之后我们为系统安装图形界面。
