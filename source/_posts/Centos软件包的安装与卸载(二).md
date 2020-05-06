---
title: Centos软件包的安装与卸载(二)
author: Evobot
abbrlink: 85ee13c4
date: 2018-04-18 23:29:28
categories: Centos7
tags:
  - Centos
image:

---



本文继续介绍关于yum的配置，如更换国内yum源以提高下载速度，和使用yum下载rpm软件包；另外将介绍如何使用源码包安装软件。

<!--more-->

---

# yum相关配置

## 更换yum源

- 由于yum配置的源默认为国外的源，访问速度较慢，所以我们可以将yum源替换为国内的源；
- 首先将上篇文章备份的`yum.repos.d`目录恢复，然后删除目录下的`CentOS7-Base.repo`文件：

```bash
[root@localhost ~]# rm -rf /etc/yum.repos.d
[root@localhost ~]# mv /etc/yum.repos.d.bak/ /etc/yum.repos.d

[root@localhost ~]# rm -f /etc/yum.repos.d/CentOS-Base.repo
```

- 然后下载网易163的yum源文件，可以使用`wget`命令或者`curl`命令，由于默认`wget`命令没有安装，并且已经删除了yum源文件，所以使用`curl -O 	http://mirrors.163.com/.help/CentOS7-Base-163.repo`下载文件：

```bash
[root@localhost ~]# cd /etc/yum.repos.d/
[root@localhost yum.repos.d]# curl -O http://mirrors.163.com/.help/CentOS7-Base-163.repo
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1572  100  1572    0     0   8253      0 --:--:-- --:--:-- --:--:--  8273
[root@localhost yum.repos.d]# ls
CentOS7-Base-163.repo  CentOS-Debuginfo.repo  CentOS-Media.repo    CentOS-Vault.repo
CentOS-CR.repo         CentOS-fasttrack.repo  CentOS-Sources.repo
```

- 然后使用命令`yum repolist`或`yum repolist all`查看新的yum源是否启用：

```bash
[root@localhost yum.repos.d]# yum repolist
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
源标识                                源名称                                             状态
base/7/x86_64                         CentOS-7 - Base - 163.com                          9,591
extras/7/x86_64                       CentOS-7 - Extras - 163.com                          448
updates/7/x86_64                      CentOS-7 - Updates - 163.com                       2,416
repolist: 12,455
```

- 接着使用`yum clean all`情况缓存，即可正常使用新的yum源，使用`yum install -y wget`安装`wget`命令。

## 安装扩展源

- 有些软件包没有在yum源内，这时需要为yum安装扩展源epel；
- 安装扩展源epel可以直接使用`yum install -y epel-release`命令，安装之后在`/etc/yum.repo.d/`目录下会下载两个epel仓库文件：

```bash
[root@evobot yum.repos.d]# ls
CentOS-Base.repo       CentOS-Epel.repo       CentOS-Sources.repo  epel-testing.repo
CentOS-CR.repo         CentOS-fasttrack.repo  CentOS-Vault.repo
CentOS-Debuginfo.repo  CentOS-Media.repo      epel.repo
```

- 再使用`yum repolist`查看epel仓库已经被启用：

```bash
[root@localhost yum.repos.d]# yum repolist 
已加载插件：fastestmirror
epel/x86_64/metalink                                                   | 6.4 kB  00:00:00     
epel                                                                   | 4.7 kB  00:00:00     
(1/3): epel/x86_64/group_gz                                            | 266 kB  00:00:01     
(2/3): epel/x86_64/updateinfo                                          | 915 kB  00:00:01     
(3/3): epel/x86_64/primary_db                                          | 6.3 MB  00:00:02     
Loading mirror speeds from cached hostfile
 * epel: mirrors.ustc.edu.cn
源标识                       源名称                                                     状态
base/7/x86_64                CentOS-7 - Base - 163.com                                   9,591
epel/x86_64                  Extra Packages for Enterprise Linux 7 - x86_64             12,491
extras/7/x86_64              CentOS-7 - Extras - 163.com                                   448
updates/7/x86_64             CentOS-7 - Updates - 163.com                                2,416
repolist: 24,946
```

## 下载rpm包

- 使用yum可以将仓库内的rpm软件包下载到本地，但下载的软件包必须是未被安装的软件包，否则会提示软件包已安装，不做处理；
- 使用`yum install -y [包名] --downloadonly`命令就可以将rpm软件包下载到本地：

```bash
[root@evobot yum.repos.d]# yum install -y zsh --downloadonly
已加载插件：fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Loading mirror speeds from cached hostfile
正在解决依赖关系
--> 正在检查事务
---> 软件包 zsh.x86_64.0.5.0.2-28.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

==============================================================================================
 Package            架构                  版本                        源                 大小
==============================================================================================
正在安装:
 zsh                x86_64                5.0.2-28.el7                os                2.4 M

事务概要
==============================================================================================
安装  1 软件包

总下载量：2.4 M
安装大小：5.6 M
Background downloading packages, then exiting:
zsh-5.0.2-28.el7.x86_64.rpm                                            | 2.4 MB  00:00:00     
exiting because "Download Only" specified
```

- 下载的软件包存放在系统`/var/cache/yum/x86_64/7/`目录下的对应的源的`Packages`目录下，软件包是从哪个源下载的，从之前的下载命令可以看到软件包的源，如上面的zsh软件包的源是`os`，所以下载的软件包在`var/cache/yum/x86_64/7/os/packages/`目录下：

```bash
[root@evobot yum.repos.d]# cd /var/cache/yum/x86_64/7/os/packages/
[root@evobot packages]# ls
zsh-5.0.2-28.el7.x86_64.rpm
```

- 也可以在下载时指定软件包下载的路径，在下载命令后使用`--downloaddir=/dirpath`选项即可：

```bash
[root@evobot packages]# yum install -y zsh --downloadonly --downloaddir=/tmp/
已加载插件：fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Loading mirror speeds from cached hostfile
正在解决依赖关系
--> 正在检查事务
---> 软件包 zsh.x86_64.0.5.0.2-28.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

==============================================================================================
 Package            架构                  版本                        源                 大小
==============================================================================================
正在安装:
 zsh                x86_64                5.0.2-28.el7                os                2.4 M

事务概要
==============================================================================================
安装  1 软件包

总下载量：2.4 M
安装大小：5.6 M
Background downloading packages, then exiting:
exiting because "Download Only" specified
[root@evobot packages]# ls /tmp/
yum_save_tx.2018-04-19.00-20.1V7tv9.yumtx  zsh-5.0.2-28.el7.x86_64.rpm
```

- 针对已安装的包想要下载rpm包，可以使用`yum reinstall`命令来下载，其余选项与前面的命令相同：

```bash
[root@evobot packages]# yum install -y vim-enhanced --downloadonly --downloaddir=/tmp
已加载插件：fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Loading mirror speeds from cached hostfile
软件包 2:vim-enhanced-7.4.160-2.el7.x86_64 已安装并且是最新版本
无须任何处理

[root@evobot packages]# yum reinstall -y vim-enhanced --downloadonly --downloaddir=/tmp
已加载插件：fastestmirror, langpacks
Repository epel is listed more than once in the configuration
Loading mirror speeds from cached hostfile
正在解决依赖关系
--> 正在检查事务
---> 软件包 vim-enhanced.x86_64.2.7.4.160-2.el7 将被 已重新安装
--> 解决依赖关系完成

依赖关系解决

==============================================================================================
 Package                  架构               版本                        源              大小
==============================================================================================
重新安装:
 vim-enhanced             x86_64             2:7.4.160-2.el7             os             1.0 M

事务概要
==============================================================================================
重新安装  1 软件包

总下载量：1.0 M
安装大小：2.2 M
Background downloading packages, then exiting:
vim-enhanced-7.4.160-2.el7.x86_64.rpm                                  | 1.0 MB  00:00:00     
exiting because "Download Only" specified

[root@evobot packages]# ls /tmp/
vim-enhanced-7.4.160-2.el7.x86_64.rpm      yum_save_tx.2018-04-19.00-23.Hu2DEg.yumtx
```

## yum源优先级

- 设置yum源的优先级，需要安装`yum-plugin-priorities`软件包；
- 确认`/etc//etc/yum/pluginconf.d/priorities.conf`文件内容如下：

```bash
[main]
enabled = 1
```

- 要在 yum 检查更新时获取权限较低的源中较新的软件，可在上面的文件中加入`check_obsoletes=1 `
- 然后在`yum.repos.d`目录下的`.repo`文件中加入`priority=N  `指定优先级，其中N为1~99，默认为99，N越小权限越高；
- 添加优先级的形式如下：

```bash
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-centos4
priority=1
#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-centos4
priority=1
[contrib]
name=CentOS-$releasever - Contrib
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=contrib
#baseurl=http://mirror.centos.org/centos/$releasever/contrib/$basearch/
gpgcheck=1
enabled=0
gpgkey=http://mirror.centos.org/centos/RPM-GPG-KEY-centos4
priority=2
```

- 官方建议的仓库优先级如下：

```bash
[base], [addons], [updates], [extras] ... priority=1
[centosplus] priority=1 (same priority as base and updates) but should be left disabled
[contrib] ... priority=2
Third Party Repos ... priority=N  (where N is > 10 and based on your preference)
```

---

# 安装源码包

- 安装源码包，需要先下载软件包的源代码，建议下载的源码包保存在`/usr/local/src`目录下；
- 这里以Apache源码包安装为例，首先下载Apache的源码包并解压：

```bash
[root@evobot src]# wget http://mirrors.cnnic.cn/apache/httpd/httpd-2.4.33.tar.gz
[root@evobot src]# tar zxvf httpd-2.4.33.tar.gz 

[root@evobot src]# cd httpd-2.4.33/
[root@evobot httpd-2.4.33]# ls
ABOUT_APACHE     BuildBin.dsp    emacs-style     LAYOUT        NOTICE            srclib
acinclude.m4     buildconf       httpd.dep       libhttpd.dep  NWGNUmakefile     support
Apache-apr2.dsw  CHANGES         httpd.dsp       libhttpd.dsp  os                test
Apache.dsw       CMakeLists.txt  httpd.mak       libhttpd.mak  README            VERSIONING
apache_probes.d  config.layout   httpd.spec      LICENSE       README.cmake
ap.d             configure       include         Makefile.in   README.platforms
build            configure.in    INSTALL         Makefile.win  ROADMAP
BuildAll.dsp     docs            InstallBin.dsp  modules       server
```

- 查看解压出的文件，一般源码包都会包含`INSTALL`和`README`文件，其中`README`一般是介绍软件包，`INSTALL`文件则是源码包安装方法；
- 在`INSTALL`中，安装Apache的步骤如下：

```bash
$ ./configure --prefix=PREFIX	# PREFIX是指安装路径，其余选项可以参考INSTALL文件
$ make
$ make install
$ PREFIX/bin/apachectl start
```

- 执行`./configure`时，如果无法判断命令执行结果是否正常，可以在运行完之后运行`echo $?`，如果返回值为`0`，则命令执行正确，否则命令执行错误；

```bash
[root@evobot httpd-2.4.33]# ./configure --prefix=/usr/local/apache
checking for chosen layout... Apache
checking for working mkdir -p... yes
checking for grep that handles long lines and -e... /bin/grep
checking for egrep... /bin/grep -E
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking target system type... x86_64-pc-linux-gnu
configure: 
configure: Configuring Apache Portable Runtime library...
configure: 
checking for APR... no
configure: error: APR not found.  Please read the documentation.
[root@evobot httpd-2.4.33]# echo $?
1
# 返回值不为0，命令执行错误，报错为APR未找到，需要下载APR
```

- Apache编译依赖APR包，所以再次下载`apr`和`apr-util`源码包解压到Apache源码包内的`srclib`目录内，另外还需要安装`gcc`、`pcre-devel.x86_64`和`expat-devel.x86_64`软件包：

```bash
[root@evobot src]# wget http://mirrors.cnnic.cn/apache/apr/apr-1.6.3.tar.gz
[root@evobot src]# wget http://mirrors.cnnic.cn/apache/apr/apr-util-1.6.1.tar.bz2
[root@evobot src]# tar zxvf apr-1.6.3.tar.gz -C httpd-2.4.33/srclib/apr
[root@evobot src]# tar jxvf apr-util-1.6.1.tar.bz2 -C httpd-2.4.33/srclib/apr-util
```

- 添加`apr`选项并重新编译安装：

```bash
[root@evobot httpd-2.4.33]# ./configure --prefix=/usr/local/apache  --with-included-apr
[root@evobot httpd-2.4.33]# make
[root@evobot httpd-2.4.33]# make install
```

- 安装完成并没有报错后，可以查看安装目录下的文件：

```bash
[root@evobot local]# ls apache/
bin  build  cgi-bin  conf  error  htdocs  icons  include  lib  logs  man  manual  modules
```

- 源码包安装相较于yum安装，可以指定安装路径，并且卸载时只需要直接删除软件安装目录即可。

---

# rpm打包

## rpm打包命令

- 将源码包打包为rpm包，可以方便将软件包移植到其他机器，方便使用；
- 在Centos7中，将源码包打包为rpm包，需要使用`yum`安装`rpmdevtools`软件包，然后执行`rpmdev-setuptree`在家目录生成rpm打包目录：

```bash
[root@localhost ~]# yum install -y rpmdevtools
[root@localhost local]# rpmdev-setuptree	# 将在家目录生成rpmbuild目录
```

- 在`rpmbuild`目录下，会有五个子目录，作用如下：

|   目录名   |         作用          |
| :-----: | :-----------------: |
|  BUILD  |     编译时所用的暂存目录      |
|  RPMS   |     存放打包好的rpm包      |
| SOURCES |     放置源码包和补丁文件      |
|  SPECS  | 放置spec模板文件，用于生成rpm包 |
|  SRPMS  |      放置rpm源码包       |

## spec文件说明

- rpm打包的关键就在于sepc文件的编写，需要为要打包的源码生成一个新的spec文件，使用命令`rpmdev-newspec [filename.spec]`生成，然后将源码包放到SOURCES目录：

```bash
[root@localhost rpmbuild]# cd SPECS/	# 进入SPEC目录
[root@localhost SPECS]# rpmdev-newspec http-2.4.33.spec	#生成spec模板
http-2.4.33.spec created; type minimal, rpm version >= 4.11.
[root@localhost SPECS]# ls
http-2.4.33.spec
[root@localhost SPECS]# cp /usr/local/src/httpd-2.4.33.tar.gz ../SOURCES/	# 复制源码包到SOURCES目录
```

- 由于Apache2.4.33依赖APR软件包，所以需要将apr和apr-util源码包也放入SOURCES目录下以便使用；
- spec文件的格式和每行作用如下：

```
Name:           http-2.4.33	# 软件名称
Version:			# 软件版本
Release:        1%{?dist}	# 发布次数
Summary:			# 软件说明
License:			# 授权模式，如GPL，即自由软件
URL:				# 源码包URL地址，可随意填写
Source0:			# 源码包名字，可以指定多个
BuildRequires:		# 编译过程依赖的工具
Requires:			# 打包生成的rpm包安装时所以来的软件包

%description		# 软件描述

%prep				# 打包准备工作，如创建目录，解压源码包等
%setup -q			# 自动解压缩源码包并进入解压出的目录

%build				# 在BUILD目录编译时的编译命令，如configure和make
%configure
make %{?_smp_mflags}

%install			# 安装到BUILDROOT虚拟目录的操作命令，如make install
rm -rf $RPM_BUILD_ROOT
%make_install

%files				# 需要添加到rpm包中的文件
%doc

%changelog			# 更新记录

# 最终生成的rpm包以{Name}-{Version}-{Release}-{BuildArch}.rpm命名
```

- 在上面的配置文件选项中，有些如`%configure`这样的字符串，这是打包定义的变量，定义变量的文件在`/usr/lib/rpm/macros`中，如`RPM_BUILD_DIR`表示`~/rpmbuild/BUILD`
- 以Apache源码包为例，配置sepc文件如下：

```bash
Name:           httpd
Version:        2.4.33
Release:        1%{?dist}
Summary:        Apache source code

License:        GPL
URL:            apache.com
Source0:        httpd-2.4.33.tar.gz

BuildRequires:  gcc
Requires:       rpm

%description


%prep
%setup -q
rm -rf srclib/apr*
tar -zxvf %_sourcedir/apr-1.6.3.tar.gz -C srclib/
tar -jxvf %_sourcedir/apr-util-1.6.1.tar.bz2 -C srclib/
pwd
mv -f srclib/apr-1.6.3 srclib/apr
mv -f srclib/apr-util-1.6.1 srclib/apr-util

%build
./configure --prefix=%_prefix  --with-included-apr
make %{?_smp_mflags}


%install
make  DESTDIR=%buildroot/usr/local/apache install


%files
%defattr(-,root,root)
/usr/local/apache

%changelog
```

- 其中的`BuildRequires`是构架rpm包时需要的依赖，`Requires`是安装软件包时的依赖包，如果构建rpm包的依赖包不存在，则会在构建时提示失败如下：

```bash
[root@localhost SPECS]# rpmbuild -ba http-2.4.33.spec 
错误：构建依赖失败：
	gcc 被 http-2.4.33-2.4.33-1.el7.centos.x86_64 需要
	automake 被 http-2.4.33-2.4.33-1.el7.centos.x86_64 需要

```

---

