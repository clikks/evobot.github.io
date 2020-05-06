---
title: LAMP架构介绍及MySQL安装
author: Evobot
categories: LAMP
tags:
  - Centos
  - MySQL
abbrlink: cd21d578
date: 2018-05-23 22:02:16
image:
---



LAMP是Linux+Apache(httpd)+MySQL+PHP几种环境组成的一种架构，很多网站运行的环境就是在LAMP的架构上运行的，Apache、MySQL和PHP可以安装在一台机器上，也可以分开安装在多台机器上，但httpd和PHP需要安装在一起。

<!--more-->

---

# LAMP架构

- httpd、PHP、MySQL三者的工资模式如下图：

  ![lamp](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/LAMP.png )

- 其中对MySQL数据库的请求是通过PHP模块进行的，这种请求是动态请求，而对于网页上的静态内容，如图片等，则是静态请求。

- 用户像网站发起请求到Apache，Apache处理用户请求，如果需要读取数据库则调用PHP模块从MySQL中查询相关的数据，而对于静态的请求则Apache会直接像用户返回静态文件数据。

---

# MySQL/Mariadb介绍

- MySQL是一个关系型数据库，最新的版本为5.7GA/8.0DMR，而5.6的版本变化较大，5.7在性能上有很大的提升；
- 而Mariadb则是MySQL被收购后由原作者发展的一个分支，Mariadb的最新版本为10.2；
- Mariadb5.5对应MySQL的5.5版本，而10.0对应MySQL的5.6版本；
- 版本划分为Community社区版本；Enterprise企业版；GA(Generally Acailable)通用版本，在生产环境中使用的；DMR(Development Milestone Release)开发里程碑发布版，表示具有重大突破的版本；RC(Release Candidate)发行候选版本；Beta开放测试版本；Alpha内部测试版本。

---

# MySQL的安装

- MySQL常用的安装包有rpm包，源码包和二进制免编译包；二进制免编译包是指已经被编译好的的安装包，使用起来比较方便；

## 下载MySQL安装包

- 一般情况下建议使用二进制免编译包，除非在需要控制性能的情况下，才需要使用源码包编译安装。
- 进入`/usr/local/src/`目录，将MySQL的二进制免编译包下载下来：`wget http://mirrors.sohu.com/mysql/MySQL-5.6/mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz`，这里下载的是5.6版本；

## 安装MySQL

- 首先解压下载下来的安装包：

  ```bash
  tar zxvf mysql-5.6.36-linux-glibc2.5-x86_64.tar.gz 
  ```

- 然后将解压出的mysql目录移动到`/usr/local/`下并重命名为mysql：

  ```bash
  mv mysql-5.6.36-linux-glibc2.5-x86_64 /usr/local/mysql
  ```

- 进入到mysql目录下，创建`mysql`用户，然后创建`/usr/local/mysql/data`目录，默认这个目录已经存在，:

  ```bash
  useradd -s /sbin/nologin -M mysql
  ```

- 然后执行下面的命令进行安装，其中`--user`指定用户，`--datadir`指定数据库存放目录：

  ```bash
  [root@evobot mysql]# ./scripts/mysql_install_db --user=mysql --datadir=/data/mysql
  ```

  - 执行这条命令有可能会报错，报错信息如下：

  ```bash
  [root@localhost mysql]# ./scripts/mysql_install_db --user=mysql --datadir=./data/mysql
  FATAL ERROR: please install the following Perl modules before executing ./scripts/mysql_install_db:
  Data::Dumper
  ```

  - 这里表示缺少Perl的模块，模块名为Dumper，我们可以使用yum配合grep搜索这个模块的软件包然后进行安装：

  ```bash
  [root@localhost mysql]# yum list | grep perl | grep -i dumper
  perl-Data-Dumper.x86_64                     2.145-3.el7                base
  perl-XML-Dumper.noarch                      0.81-17.el7                base
  ```

  - 这里由于不清楚包名的大小写，所以使用`grep -i`不区分大小写进行过滤；
  - 搜索出来的包，我们可以进行尝试安装，然后再重新执行mysql的安装命令确认是否安装了正确的依赖包，如果不想一个一个尝试安装，也可以将搜索出来的包全部安装，这里安装`perl-Data-Dumper.x86_64`，然后重新执行mysql安装脚本。

- 安装完之后验证是否正确安装可以执行`echo $?`或者查看安装时的输出是否有两个`OK`。

## 复制配置文件及启动脚本

- mysql的模板配置文件为`/usr/local/mysql/support-files/my-default.cnf`，将其复制到`/etc/`下并重命名为`my.cnf`：

  ```bash
  [root@localhost mysql]# cp support-files/my-default.cnf /etc/my.cnf
  ```

  - 实际上在`/etc/`下存在一个my.cnf的配置文件，使用`rpm -qf`查看该文件来自哪个软件包，可以看到是由mariadb-libs安装到系统内的：

  ```bash
  [root@evobot mysql]# rpm -qf /etc/my.cnf
  mariadb-libs-5.5.56-2.el7.x86_64
  ```

  - 这个配置文件也可以直接使用，但是需要更改其中的相关配置，如datadir更改为`/usr/local/mysql/data/mysql`，socket更改为`/tmp/mysql.sock`，并且注释`mysq_safe`下的配置项。

- mysql的启动脚本模板文件为`/usr/local/mysql/support-files/mysql.server`，将其复制到`/etc/init.d/`,并重命名为`mysqld`，**在替换过配置文件后，需要重新执行初始化数据库的命令`mysql_install_db`**:

  ```bash
  [root@localhost mysql]# cp support-files/mysql.server /etc/init.d/mysqld
  ```

- 然后修改启动脚本中的`basedir`和`datadir`的值分别为`/usr/local/mysql`和`/data/mysql`：

  ```bash
  # If you change base dir, you must also change datadir. These may get
  # overwritten by settings in the MySQL configuration files.
  
  basedir=/usr/local/mysql
  datadir=/data/mysql
  ```

- 修改启动脚本的权限为755：

  ```bash
  [root@localhost mysql]# chmod 755 /etc/init.d/mysqld
  [root@localhost mysql]# ls -l /etc/init.d/mysqld
  -rwxr-xr-x. 1 root root 10592 5月  23 23:54 /etc/init.d/mysqld
  ```

- 设置MySQL为开机启动，使用`chkconfig --add mysqld`命令：

  ```bash
  [root@localhost mysql]# chkconfig --add mysqld
  [root@localhost mysql]# chkconfig --list
  
  注意：该输出结果只显示 SysV 服务，并不包含原生 systemd 服务。SysV 配置数据可能被原生 systemd 配置覆盖。
        如果您想列出 systemd 服务,请执行 'systemctl list-unit-files'。
        欲查看对特定 target 启用的服务请执行
        'systemctl list-dependencies [target]'。
  
  mysqld          0:关    1:关    2:开    3:开    4:开    5:开    6:关
  netconsole      0:关    1:关    2:关    3:关    4:关    5:关    6:关
  network         0:关    1:关    2:开    3:开    4:开    5:开    6:关
  ```

## 启动MySQL

- 启动MySQL服务，使用`/etc/init.d/mysqld start`：

  ```bash
  [root@evobot mysql]# /etc/init.d/mysqld start
  Starting MySQL.Logging to '/usr/local/mysql/data/mysql/evobot.err'.
   SUCCESS!
  ```

  - 也可以使用`service mysqld start`进行启动，手动启动MySQL的另一种方式如下，这种方式可以指定MySQL的配置文件，数据库目录和用户：

  ```bash
  [root@evobot mysql]# /usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf --user=mysql --datadir=/usr/local/mysql/data/mysql &
  [1] 31732
  [root@evobot mysql]# 180524 00:29:11 mysqld_safe Logging to '/usr/local/mysql/data/mysql/evobot.err'.
  180524 00:29:11 mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/data/mysql
  ```

  - 手动运行MySQL，需要停止进程，则使用`killall mysqld`结束进程，不建议使用`kill`命令，因为MySQL运行时可能在读写数据，如果使用`kill`，那么会造成数据丢失，而`killall`则会先等待数据读写完成再杀死进程。

- 查看MySQL运行时的进程：

  ```bash
  [root@evobot mysql]# !ps
  ps aux| grep mysql
  root     31732  0.0  0.0 113260  1608 pts/0    S    00:29   0:00 /bin/sh /usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my.cnf --user=mysql --datadir=/usr/local/mysql/data/mysql
  mysql    31864  6.3 24.0 1304348 452568 pts/0  Sl   00:29   0:00 /usr/local/mysql/bin/mysqd --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data/mysql --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/usr/local/mysql/data/mysql/evobot.err --pid-file=/usr/local/mysql/data/mysql/evobot.pid --socket=/tmp/mysql.sock
  root     31911  0.0  0.0 112676   980 pts/0    R+   00:29   0:00 grep --color=auto mysql
  ```

- MySQL的默认监听端口为3306：

  ```bash
  [root@evobot mysql]# netstat -tlnp
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
  tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      1/systemd
  tcp        0      0 0.0.0.0:2233            0.0.0.0:*               LISTEN      801/sshd
  tcp6       0      0 :::3306                 :::*                    LISTEN      31864/mysqld
  ```

- MySQL的引擎由`innodb`和`myisam`，`myisam`较为轻量。


# MySQL5.5编译安装

- 首先下载MySQL5.5.60源码包，[MySQL5.5.60下载地址](https://dev.mysql.com/get/Downloads/MySQL-5.5/mysql-5.5.60.tar.gz);

- 解压源码包，创建mysql用户，然后安装编译需要的依赖包：
  
  ```bash
  tar zxvf mysql-5.5.60.tar.gz
  
  cd mysql-5.5.60
  
  useradd -s /sbin/nologin -M mysql
  
  yum install -y cmake gcc gcc-c++ ncurses-devel bison
  ```
  > 如果编译过程中因为缺少依赖包出错，再安装了依赖包后，需要执行`rm -f CMakeCache.txt`删除编译缓存文件后再重新进行编译。
  
- 然后进行编译操作，默认情况下，mysql安装目录为`/usr/local/mysql`，数据目录为`/usr/local/mysql/data`，编译参数可以参照[MySQL Source-Configuration Options](https://dev.mysql.com/doc/refman/5.5/en/source-configuration-options.html)，编译过程如下：

  ```bash
  cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql \
  -DMYSQL_DATADIR=/mysql/data \
  -DDEFAULT_CHARSET=utf8 \
  -DDEFAULT_COLLATION=utf8_general_ci \
  -DWITH_EXTRA_CHARSETS:STRING=all \
  -DWITH_DEBUG=0 -DWITH_SSL=yes \
  -DWITH_READLINE=1 \
  -DENABLE_LOCAL_INFILE=1
  
  make && make install
  ```

- 编译完成后，进入安装目录，更改目录属主和属组，然后进行初始化安装，最后复制配置文件和启动脚本，执行下面的命令完成MySQL的安装：

  ```bash
  cd /usr/local/mysql
  
  ./scripts/mysql_install_db --user=mysql --datadir=/mysql/data
  
  cp support-files/my-medium.cnf /etc/my.cnf
  
  cp support-files/mysql.server /etc/init.d/mysqld
  ```
  
- 最后，使用`/etc/init.d/mysqld start`即可启动MySQL服务。

# MySQL5.7二进制包安装
## 安装

- 首先下载MySQL5.7.17二进制安装包，[下载链接](http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz)；

- 然后解压二进制包，并将解压出来的目录移动到/usr/local目录下更名为mysql:

  ```bash
  tar zxvf mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz
  
  mv mysql-5.7.17-linux-glibc2.5-x86_64 /usr/local/mysql
  ```

- 接着使用yum安装MySQL5.7的依赖包`perl-Data-Dumper`、 `libaio-devel`、 `numactl`、 `gcc`、 `openssl`;

- 然后进行初始化操作：
  
  ```bash
  mkdir -p /data/mysql
  
  cd /usr/local/mysql/
  
  ./bin/mysqld --initialize --user=mysql --datadir=/data/mysql/
  ```
  >这里成功执行后，会再最后一行打印MySQL的root账户的密码：
  >`[Note] A temporary password is generated for root@localhost: CrG_3SlTYe3:`
  
- 然后生成密钥：
  
  ```bash
  ./bin/mysql_ssl_rsa_setup --datadir=/data/mysql/
  ```

- 复制配置文件和启动脚本：

  ```bash
  cp support-files/my-default.cnf /etc/my.cnf
  
  cp support-files/mysql.server /etc/init.d/mysqld
  ```

- 接着修改`/etc/my.cnf`，配置basedir、datadir、port以及socket：

  ```bash
  basedir = /usr/local/mysql
  datadir = /data/mysql
  port = 3306
  socket = /tmp/mysql.sock
  ```
  
- 修改`/etc/init.d/mysqld`启动脚本，配置basedir和datadir：

  ```bash
  basedir=/usr/local/mysql
  datadir=/data/mysql
  ```

- 完成后，即可使用`/etc/init.d/mysqld start`启动MySQL。

## 修改root密码

- 之前初始化安装时已经给出了root的密码，使用密码就能够登陆MySQL，登陆后需要重新修改密码：
  
  ```bash
  /usr/local/mysql/bin/mysql -uroot -pCrG_3SlTYe3:
  
  //进入MySQL修改密码
  mysql> set password=password('123456');
  ```
  
- 如果忘记了初始化安装时的密码，可以修改my.cnf配置，在[mysqld]下面增加一行`skip-grant-tables`,然后重启MySQL服务，执行`/usr/local/mysql/bin/mysql -uroot`无密码登陆进MySQL中，然后在MySQL命令行内执行下面的命令：

  ```bash
  mysql> update mysql.user set authentication_string=password('112233') where user='root';
  ```

- 完成后，将my.cnf中的`skip-grant-tables`配置删除，重启MySQL服务即可使用新的密码登陆。
---



  