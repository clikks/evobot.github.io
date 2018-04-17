---
title: Centos软件包的安装与卸载(一)
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 2adf88a6
date: 2018-04-16 21:20:45
image:
---



本文主要介绍Centos中安装软件包的3种方法，以及rpm包、rpm工具的使用方法和yum命令如何安装软件包。

在Centos中安装软件有三种方式，分别是以下三种：

1. rpm工具
   - rpm软件包类似windows下的exe文件，其安装路径等是已经打包固定的；
2. yum工具
   - yum工具是使用python开发的软件包安装工具，yum能够自动解决软件包依赖问题，实际上yum安装的软件包仍然是rpm包；
3. 源码包
   - 源码包是直接将编程语言编写的软件打包，安装时需要使用对应的编译工具对源代码进行编译安装。

<!--more-->

---

# rpm工具

## rpm包格式

- 在虚拟机中，需要将Centos的安装光盘进行挂载，光盘的设备文件是`/dev/sr0`，也可以使用`/dev/cdrom`，其是`/dev/sr0`的软链接文件；
- 使用`mount /dev/cdrom /mnt`将光盘挂载到`/mnt`目录，进入目录，查看`Packages`目录下的文件，该目录下就是`rpm`软件包：

```bash
[root@localhost Packages]# ls
389-ds-base-1.3.5.10-11.el7.x86_64.rpm
389-ds-base-libs-1.3.5.10-11.el7.x86_64.rpm
abattis-cantarell-fonts-0.0.16-3.el7.noarch.rpm
abrt-2.1.11-45.el7.centos.x86_64.rpm
abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64.rpm
abrt-addon-kerneloops-2.1.11-45.el7.centos.x86_64.rpm
abrt-addon-pstoreoops-2.1.11-45.el7.centos.x86_64.rpm
abrt-addon-python-2.1.11-45.el7.centos.x86_64.rpm
abrt-addon-vmcore-2.1.11-45.el7.centos.x86_64.rpm
abrt-addon-xorg-2.1.11-45.el7.centos.x86_64.rpm
```

- `rpm`包的文件名格式是使用`-`分隔，其中第一段字符组成的是软件包包名；第二段是软件包的版本号，使用`.`分隔，又分为主版本、次版本和小版本；第三段则是发布版本号，代表了软件包对应的操作系统版本，如`el7`对应了Centos7；最后一段是平台位数，分为32位和64位，Centos7不再区分32、64位，只有64位软件包，但64位系统能够安装32位的软件包；

## rpm工具用法

- 安装`rpm`包，使用`rpm -ivh [rpm包文件]`命令，选项`-i`表示`install`，`-v`表示可视化，`-h`表示人性化显示：

```bash
[root@localhost Packages]# rpm -ivh zsh-5.0.2-25.el7.x86_64.rpm 
准备中...                          ################################# [100%]
正在升级/安装...
   1:zsh-5.0.2-25.el7                 ################################# [100%]
```

- 对于已经安装的软件包需要升级操作，则使用`rpm -Uvh [rpm包文件]`，选项`-U`表示`update`：

```bash
[root@localhost Packages]# rpm -Uvh zsh-5.0.2-25.el7.x86_64.rpm 
准备中...                          ################################# [100%]
	软件包 zsh-5.0.2-25.el7.x86_64 已经安装
```

- 卸载已安装的软件包，则使用`rpm -e [包名]`进行卸载，正常卸载时，执行命令无输出，很多时候卸载一个软件，会提示软件包被依赖无法卸载，这时需要先卸载依赖要卸载的软件包的包：

```bash
[root@localhost Packages]# rpm -e zsh
[root@localhost Packages]# 

[root@localhost Packages]# rpm -e ppp
错误：依赖检测失败：
	ppp = 2.4.5 被 (已安裝) NetworkManager-1:1.4.0-12.el7.x86_64 需要
[root@localhost Packages]# rpm -e NetworkManager
错误：依赖检测失败：
	NetworkManager = 1:1.4.0-12.el7 被 (已安裝) NetworkManager-tui-1:1.4.0-12.el7.x86_64 需要
	NetworkManager(x86-64) = 1:1.4.0-12.el7 被 (已安裝) NetworkManager-team-1:1.4.0-12.el7.x86_64 需要
	NetworkManager(x86-64) = 1:1.4.0-12.el7 被 (已安裝) NetworkManager-wifi-1:1.4.0-12.el7.x86_64 需要
```

- 查询已经安装的包，使用`rpm -qa`，执行命令后，会将系统所以已经安装的包列出：

```bash
[root@localhost Packages]# rpm -qa
grub2-2.02-0.44.el7.centos.x86_64
centos-release-7-3.1611.el7.centos.x86_64
openssh-server-6.6.1p1-31.el7.x86_64
filesystem-3.2-21.el7.x86_64
authconfig-6.2.8-14.el7.x86_64
ncurses-base-5.9-13.20130511.el7.noarch
audit-2.6.5-3.el7.x86_64
bind-license-9.9.4-37.el7.noarch
aic94xx-firmware-30-6.el7.noarch
...
```

- 查询单独的一个rpm包，使用`rpm -q [包名]`进行查询：

```bash
[root@localhost Packages]# rpm -q ppa
未安装软件包 ppa 
[root@localhost Packages]# rpm -q ppp
ppp-2.4.5-33.el7.x86_64
```

- 上面的查询并未列出软件包的详细信息，可以使用`rpm -qi [包名]`查询软件包的详细信息：

```bash
[root@localhost Packages]# rpm -qi ppp
Name        : ppp
Version     : 2.4.5
Release     : 33.el7
Architecture: x86_64
Install Date: 2018年04月09日 星期一 21时22分32秒
Group       : System Environment/Daemons
Size        : 872624
License     : BSD and LGPLv2+ and GPLv2+ and Public Domain
Signature   : RSA/SHA256, 2014年07月04日 星期五 12时34分15秒, Key ID 24c6a8a7f4a80eb5
Source RPM  : ppp-2.4.5-33.el7.src.rpm
Build Date  : 2014年06月10日 星期二 14时27分03秒
Build Host  : worker1.bsys.centos.org
Relocations : (not relocatable)
Packager    : CentOS BuildSystem <http://bugs.centos.org>
Vendor      : CentOS
URL         : http://www.samba.org/ppp
Summary     : The Point-to-Point Protocol daemon
Description :
The ppp package contains the PPP (Point-to-Point Protocol) daemon and
documentation for PPP support. The PPP protocol provides a method for
transmitting datagrams over serial point-to-point links. PPP is
usually used to dial in to an ISP (Internet Service Provider) or other
organization over a modem and phone line.
```

- 命令`rpm -ql [包名]`可以列出软件包安装时所包含的文件：

```bash
[root@localhost Packages]# rpm -ql vim-enhanced
/etc/profile.d/vim.csh
/etc/profile.d/vim.sh
/usr/bin/rvim
/usr/bin/vim
/usr/bin/vimdiff
/usr/bin/vimtutor
```

- 而命令`rpm -qf [文件绝对路径]`则可以查询文件是由哪个软件包安装:

```bash
[root@localhost Packages]# rpm -qf /usr/bin/vim
vim-enhanced-7.4.160-2.el7.x86_64

[root@localhost Packages]# rpm -qf `which cd`	#使用`which cd`查询命令绝对路径，然后rpm -qf查询软件包
bash-4.2.46-20.el7_2.x86_64

```

---

# yum工具

## yum介绍

- rpm在安装软件包时也会存在依赖问题：

```bash
[root@localhost Packages]# rpm -ivh abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64.rpm 
错误：依赖检测失败：
	abrt = 2.1.11-45.el7.centos 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
	abrt-libs = 2.1.11-45.el7.centos 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
	abrt-retrace-client 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
	elfutils 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
	gdb >= 7.6.1-63 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
	libabrt.so.0()(64bit) 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
	libreport-python 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
	libreport.so.0()(64bit) 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
	libsatyr.so.3()(64bit) 被 abrt-addon-ccpp-2.1.11-45.el7.centos.x86_64 需要
```

- `yum`为了解决安装软件包的依赖问题而产生的软件包安装工具，`yum`可以自动安装软件包所依赖的包，并且可以从网络软件仓库安装所需要的软件包；

## yum的用法

- `yum list`可以列出能够安装的软件包，列出的软件包信息中，最左侧是包名，中间是版本号，最右边是仓库名：

```bash
[root@localhost Packages]# yum list
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.sohu.com
 * extras: mirrors.sohu.com
 * updates: mirrors.aliyun.com
已安装的软件包
GeoIP.x86_64                                                  1.5.0-11.el7                              @anaconda
NetworkManager.x86_64                                         1:1.4.0-12.el7                            @anaconda
NetworkManager-libnm.x86_64                                   1:1.4.0-12.el7                            @anaconda
NetworkManager-team.x86_64                                    1:1.4.0-12.el7                            @anaconda
NetworkManager-tui.x86_64                                     1:1.4.0-12.el7                            @anaconda
NetworkManager-wifi.x86_64                                    1:1.4.0-12.el7                            @anaconda
acl.x86_64                                                    2.2.51-12.el7                             @anaconda
aic94xx-firmware.noarch                                       30-6.el7                                  @anaconda
...
```

- `yum`的仓库名配置在`/etc/yum.repos.d/`目录下，其中`Centos-Base.repo`就是`base`仓库的配置文件：

```bash
[root@localhost Packages]# ls /etc/yum.repos.d/
CentOS-Base.repo  CentOS-Debuginfo.repo  CentOS-Media.repo    CentOS-Vault.repo
CentOS-CR.repo    CentOS-fasttrack.repo  CentOS-Sources.repo

CentOS-CR.repo    CentOS-fasttrack.repo  CentOS-Sources.repo
[root@localhost Packages]# cat /etc/yum.repos.d/CentOS-Base.repo 
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

```

- 使用`yum`搜索一个软件包，使用`yum search [包名关键字]`，使用这种方法搜索，会列出包名和包说明中含有关键字的包，搜索不够精确：

```bash
[root@localhost Packages]# yum search note
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.sohu.com
 * extras: mirrors.sohu.com
 * updates: mirrors.aliyun.com
=============================================== N/S matched: note ===============================================
gnote.i686 : Note-taking application
gnote.x86_64 : Note-taking application
texlive-marginnote.noarch : Notes in the margin, even where \marginpar fails
texlive-marginnote-doc.noarch : Documentation for marginnote
bltk.i686 : The BLTK measures notebook battery life under any workload
bltk.x86_64 : The BLTK measures notebook battery life under any workload
perl-HTML-FormatText-WithLinks.noarch : HTML to text conversion with links as footnotes
texlive-bigfoot.noarch : Footnotes for critical editions
texlive-footmisc.noarch : A range of footnote options
texlive-threeparttable.noarch : Tables with captions and notes all the same width

  名称和简介匹配 only，使用“search all”试试。
```

- 想要更精准的搜索软件包，使用`yum list | grep [包名]`进行搜索，`grep`会将匹配到的包名过滤出来：

```bash
[root@localhost Packages]# yum list | grep 'vim'
vim-common.x86_64                           2:7.4.160-2.el7            @base    
vim-enhanced.x86_64                         2:7.4.160-2.el7            @base    
vim-filesystem.x86_64                       2:7.4.160-2.el7            @base    
vim-minimal.x86_64                          2:7.4.160-1.el7            @anaconda
protobuf-vim.x86_64                         2.5.0-8.el7                base     
vim-X11.x86_64                              2:7.4.160-2.el7            base     
vim-minimal.x86_64                          2:7.4.160-2.el7            base     
```

## 安装与卸载

- 使用`yum`安装软件把，命令为`yum install [-y] [包名]`，其中`-y`是确认安装，否则在安装时会询问是否安装软件包；
- `yum grouplist`可以查看软件包组，软件包组就是在安装系统时所选择的安装方式，如最小化安装：

```bash
[root@localhost Packages]# yum grouplist
已加载插件：fastestmirror
没有安装组信息文件
Maybe run: yum groups mark convert (see man yum)
Loading mirror speeds from cached hostfile
 * base: mirrors.sohu.com
 * extras: mirrors.sohu.com
 * updates: mirrors.aliyun.com
可用的环境分组：
   最小安装
   基础设施服务器
   计算节点
   文件及打印服务器
   基本网页服务器
   虚拟化主机
   带 GUI 的服务器
   GNOME 桌面
   KDE Plasma Workspaces
   开发及生成工作站
可用组：
   传统 UNIX 兼容性
   兼容性程序库
   图形管理工具
   安全性工具
   开发工具
   控制台互联网工具
   智能卡支持
   科学记数法支持
   系统管理
   系统管理工具
完成
```

- 想要安装软件包组的软件，则使用`yum groupinstall [-y] [软件包组] `，软件包组需要使用英文，改变系统语言环境，使用`LANG=en`：
- 卸载软件包使用命令`yum remove [-y] [包名]`,使用`yum`卸载软件时，会将软件包的依赖包同时卸载：

```bash
[root@localhost Packages]# yum remove ppp
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Resolving Dependencies
--> Running transaction check
---> Package ppp.x86_64 0:2.4.5-33.el7 will be erased
--> Processing Dependency: ppp = 2.4.5 for package: 1:NetworkManager-1.4.0-12.el7.x86_64
--> Running transaction check
---> Package NetworkManager.x86_64 1:1.4.0-12.el7 will be erased
--> Processing Dependency: NetworkManager = 1:1.4.0-12.el7 for package: 1:NetworkManager-tui-1.4.0-12.el7.x86_64
--> Processing Dependency: NetworkManager(x86-64) = 1:1.4.0-12.el7 for package: 1:NetworkManager-team-1.4.0-12.el7.x86_64
--> Processing Dependency: NetworkManager(x86-64) = 1:1.4.0-12.el7 for package: 1:NetworkManager-wifi-1.4.0-12.el7.x86_64
--> Running transaction check
---> Package NetworkManager-team.x86_64 1:1.4.0-12.el7 will be erased
---> Package NetworkManager-tui.x86_64 1:1.4.0-12.el7 will be erased
---> Package NetworkManager-wifi.x86_64 1:1.4.0-12.el7 will be erased
--> Finished Dependency Resolution

Dependencies Resolved

=================================================================================================================
 Package                          Arch                Version                       Repository              Size
=================================================================================================================
Removing:
 ppp                              x86_64              2.4.5-33.el7                  @anaconda              852 k
Removing for dependencies:
 NetworkManager                   x86_64              1:1.4.0-12.el7                @anaconda               10 M
 NetworkManager-team              x86_64              1:1.4.0-12.el7                @anaconda               53 k
 NetworkManager-tui               x86_64              1:1.4.0-12.el7                @anaconda              266 k
 NetworkManager-wifi              x86_64              1:1.4.0-12.el7                @anaconda              144 k

Transaction Summary
=================================================================================================================
Remove  1 Package (+4 Dependent packages)

Installed size: 11 M
Is this ok [y/N]: N
Exiting on user command
Your transaction was saved, rerun it with:
 yum load-transaction /tmp/yum_save_tx.2018-04-16.23-01.7GYVHH.yumtx
```

- 使用`yum`升级软件包，命令为`yum update [-y] [包名]`，如果不加包名升级，则会自动升级系统中所以可以升级的包，一般在刚刚安装完系统时执行，如果系统中已经运行有服务，升级可能会造成软件包的不兼容；
- `yum provides [命令路径]`可以查询命令是哪个软件包提供的，命令路径可以使用通配符，如`yum provides "/*/vim"`，使用这种方法的前提是系统中没有安装对应的软件包，在实际使用中可以配合`rpm -qf`对命令进行搜索，再使用`yum provides`进行搜索。

# 搭建yum仓库

## yum本地仓库搭建

- yum默认的仓库需要链接远程服务器下载资源，但某些情况下存在服务器无法联网的情况，所以可以在本地搭建yum仓库；
- 搭载本地仓库需要挂载光盘或者光盘镜像，在上文中已经将光盘镜像挂载到`/mnt`目录；
- 将`/etc/yum.repos.d/`目录备份，使用命令`cp -r /etc/yum.repos.d /etc/yum.repos.d.bak`进行备份操作，然后将原`/etc/yum.repos.d`目录清空：

```bash
[root@localhost ~]# cp -r /etc/yum.repos.d/ /etc/yum.repos.d.bak
[root@localhost ~]# cd /etc/yum.repos.d
[root@localhost yum.repos.d]# ls
CentOS-Base.repo  CentOS-Debuginfo.repo  CentOS-Media.repo    CentOS-Vault.repo
CentOS-CR.repo    CentOS-fasttrack.repo  CentOS-Sources.repo
[root@localhost yum.repos.d]# rm -f *
```

- 在`/etc/yum.repos.d`目录下创建`dvd.repo`文件，写入以下内容：

```bash
[dvd]
name=install dvd	
baseurl=file:///mnt	
enable=1		
gpgcheck=0
```

- **name**字段是对仓库名的描述，**baseurl**是仓库的路径，**enable**表示是否启用仓库，**gpgcheck**表示是否进行gpg检查，由于是本地仓库，所以置为0；
- 由于之前使用远程仓库，系统存在缓存，所以执行`yum clean all`命令清除缓存，再执行`yum list`就可以看到软件包的仓库已经变为`dvd`

```bash
[root@localhost yum.repos.d]# yum clean all
已加载插件：fastestmirror
正在清理软件源： dvd
Cleaning up everything
Cleaning up list of fastest mirrors

[[root@localhost yum.repos.d]# yum list
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
已安装的软件包
GeoIP.x86_64                                     1.5.0-11.el7                        @anaconda
bzip2.x86_64                                     1.0.6-13.el7                        @base    
centos-bookmarks.noarch                          7-1.el7                             dvd     
```

- `yum list`列出的软件包，在第三列有3种显示：`@anaconda`、`@base`表示已经安装过的软件包，`dvd`则是来自我们本地搭建的仓库dvd的软件包，之后使用`yum`安装软件包时，就会从本地的仓库进行安装。

## yum局域网仓库搭建

### FTP服务搭建

- 在局域网中搭建yum仓库，需要在一台服务器上搭建`Apache`服务器或者`ftp`服务器，这里使用`vsftpd`软件包搭建`ftp`服务器做yum仓库；
- 使用`yum install -y vsftpd`安装vsftp软件包，然后执行`systemctl enable vsftpd`命令将vsftp服务设置为开机启动，之后使用`systemctl start vsftpd`命令启动服务：

```bash
[root@localhost yum.repos.d]# systemctl enable vsftpd
Created symlink from /etc/systemd/system/multi-user.target.wants/vsftpd.service to /usr/lib/systemd/system/vsftpd.service.

[root@localhost yum.repos.d]# systemctl start vsftpd
```

- 命令`systemctl is-enabled vsftpd`可以查看服务是否被设置为开机启动，`enable`为开机启动，`disable`为不启动；
- 启动vsftp服务后，还需要关闭防火墙，否则无法访问FTP服务，使用命令`systemctl stop firewalld.service`关闭防火墙，如果想要禁止防火墙启动，执行`systemctl disable firewalld.service`即可；
- 这时从浏览器使用`ftp://[ip address]`地址访问FTP服务器，出现下面的界面表示FTP服务运行正常：

![FTP](http://p5qynomrl.bkt.clouddn.com/15239803743973aqlnr8b.png?imageslim)

### 仓库设置

- 在`/var/ftp/pub/`目录下创建centos7目录，将Centos7的安装光盘镜像内的文件复制到目录下：

```bash
[root@localhost ~]# mkdir /var/ftp/pub/centos7
[root@localhost ~]# cp -rvf /mnt/* /var/ftp/pub/centos7/
```

- 复制完成后，在浏览器中访问FTP，进入`centos7/Packages`目录，就可以看到所有的rpm软件包：

![rpm](http://p5qynomrl.bkt.clouddn.com/15239832162285ykeole1.png?imageslim)

- 接着安装`createrepo`软件包用来身材yum仓库的rpm元数据，使用`yum install -y createrepo`安装；
- 安装完成后，使用命令`createrepo /var/ftp/pub/centos7`生成rpm元数据：

```bash
[root@localhost ~]# createrepo /var/ftp/pub/centos7/
Spawning worker 0 with 3831 pkgs
Workers Finished
Saving Primary metadata
Saving file lists metadata
Saving other metadata
Generating sqlite DBs
Sqlite DBs complete
```

### 使用软件源

- 软件源仓库设置好之后，在其他机器上进行yum软件源的配置，首先与搭建本地仓库相同，将系统`/etc/yum.repos.d`目录备份；
- 备份完成后，删除`yum.repos.d`除`CentOS-Base.repo`文件外的其他文件：

```bash
[root@localhost yum.repos.d]# ls
CentOS-Base.repo  CentOS-Debuginfo.repo  CentOS-Media.repo    CentOS-Vault.repo
CentOS-CR.repo    CentOS-fasttrack.repo  CentOS-Sources.repo

[root@localhost yum.repos.d]# find . ! -name "*Base*" -delete	# !表示查找除"*Base*"文件外的文件，-delete表示删除查找到的文件
[root@localhost yum.repos.d]# ls
CentOS-Base.repo
```

- 然后修改`CentOS-Base.repo`文件，修改后的内容如下：

```bash
[base]
name=CentOS-$releasever - Base
baseurl=ftp://192.168.199.224/pub/centos7/	
# 将IP地址替换为FTP服务器地址
gpgcheck=1
gpgkey=ftp://192.168.199.224/pub/centos7/RPM-GPG-KEY-CentOS-7
enable=1
#released updates
#[updates]
#name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
#gpgcheck=1
#gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
#enable=1
```

- 保存之后的操作与本地仓库相同，执行`yum clean all`和`yum update`，之后就可以使用局域网软件源了。

---

# 保存yum下载的rpm包

- 修改相应的配置文件可以保存每次使用`yum`安装的软件的rpm软件包；
- 修改`/etc/yum.conf`文件，修改内容如下：

```bash
[main]
# cachedir是保存下载的包的目录，可以修改为自己定义的位置
cachedir=/var/cache/yum/$basearch/$releasever
# keepcache为1时表示保存已下载的包
keepcache=1
debuglevel=2
logfile=/var/log/yum.log
exactarch=1
obsoletes=1
gpgcheck=1
plugins=1
installonly_limit=5
bugtracker_url=http://bugs.centos.org/set_project.php?project_id=23&ref=http://bugs.centos.org/bug_report_page.php?category=yum
distroverpkg=centos-release
```

---



