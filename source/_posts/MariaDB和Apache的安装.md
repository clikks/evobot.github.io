---
title: MariaDB和Apache的安装
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
  - Apache
  - MariaDB
abbrlink: 1d2820c6
date: 2018-05-24 23:02:55
image:
---



MariaDB由于与MySQL基本相同，所以安装方法也基本一致，这里介绍MariaDB的免编译二进制安装包的安装方法，另外也将介绍Apache的安装方法。

<!--more-->

---

# MariaDB安装

- 下载MariaDB：

  ```bash
  # cd /usr/local/src
  # wget https://downloads.mariadb.com/MariaDB/mariadb-10.2.6/bintar-linux-glibc_214-x86_64/mariadb-10.2.6-linux-glibc_214-x86_64.tar.gz
  ```

- 解压MariaDB压缩包：

  ```bash
  # tar zxvf mariadb-10.2.6-linux-glibc_214-x86_64.tar.gz
  ```

- 将解压出来的目录移动到`/usr/local/`下并更名为`mariadb`：

  ```bash
  # mv mariadb-10.2.6-linux-glibc_214-x86_64 /usr/local/mariadb
  ```

- 创建`mysql`用户并创建`/data/mariadb`目录，或使用mariadb目录下的data目录：

  ```bash
  # useradd mysql -s /bin/nologin
  # mkdir -p /data/mariadb
  ```

- 执行初始化脚本：

  ```bash
  # ./scripts/mysql_install_db --user=mysql --datadir=/data/mariadb/
  ```

  - 如果在执行初始化脚本是提示如下错误，那么需要将`/etc/my,cnf`删除或者重命名，这是由于之前安装了MySQL并且修改了`my.cnf`内的basedir等配置造成的，或者在初始化脚本的参数上增加`--basedir=/usr/local/mariadb`：

  ```bash
  FATAL ERROR: Could not find mysqld
  
  The following directories were searched:
  
      /usr/local/mysql/libexec
      /usr/local/mysql/sbin
      /usr/local/mysql/bin
  
  If you compiled from source, you need to either run 'make install' to
  copy the software into the correct location ready for operation.
  If you don't want to do a full install, you can use the --srcddir
  option to only install the mysql database and privilege tables
  
  If you are using a binary release, you must either be at the top
  level of the extracted archive, or pass the --basedir option
  pointing to that location.
  
  The latest information about mysql_install_db is available at
  ```

- 复制配置文件和启动脚本：

  MariaDB提供了不同的配置文件，如my-small.cnf，my-large.cnf，这些配置文件的不同之处在于其定义的MariaDB运行时的缓存，缓冲等大小不一样，可以根据自己机器的性能和配置，选择合适的配置文件复制：

  另外为了与MySQL的配置做区分，这里不在复制到`/etc/`下，而是复制到MariaDB的basedir下：

  ```bash
  # cp support-files/my-small.cnf /usr/local/mariadb/my.cnf
  # cp support-files/mysql.server /etc/init.d/mariadb
  ```

- 编辑启动脚本和配置文件

  - 为了防止MariaDB启动时由于未配置datadir，导致启动时MariaDB读取`/etc/my.cnf`中配置的datadir，所以需要在MariaDB自定义的配置文件中增加datadir：

  ```bash
  [mysqld]
  datadir=/data/mariadb
  ```

  - MariaDB的启动脚本需要对`basedir`、`datadir`进行修改，并且增加`conf`配置，然后再修改启动命令，指定配置文件启动：

  ```bash
  basedir=/usr/local/mariadb
  datadir=/data/mariadb
  conf=$basedir/my.cnf
  ...
  case "$mode" in
    'start')
      # Start daemon
  
      # Safeguard (relative paths, core dumps..)
      cd $basedir
  
      echo $echo_n "Starting MySQL"
      if test -x $bindir/mysqld_safe
      then
        # Give extra arguments to mysqld with the my.cnf file. This script
        # may be overwritten at next upgrade.
        # 增加--defaults-file选项
        $bindir/mysqld_safe --defaults-file="$conf" --datadir="$datadir" --pid-file="$mysqld_pid_file_path" "$@" &
        wait_for_ready; return_value=$?
  ...
  ```

- 启动服务

  ```bash
  [root@localhost mariadb]# /etc/init.d/mariadb start
  Reloading systemd:                                         [  确定  ]
  Starting mariadb (via systemctl):                          [  确定  ]
  
  [root@localhost mariadb]# ps aux | grep mysql
  root       2288  0.0  0.1 115380  1736 ?        S    00:19   0:00 /bin/sh /usr/local/mariadb/bin/mysqld_safe --defaults-file=/usr/local/mariadb/my.cnf --datadir=/data/mariadb --pid-file=/data/mariadb/localhost.localdomain.pid
  mysql      2404  0.3  5.6 1583772 56676 ?       Sl   00:19   0:00 /usr/local/mariadb/bin/mysqld --defaults-file=/usr/local/mariadb/my.cnf --basedir=/usr/local/mariadb --datadir=/data/mariadb --plugin-dir=/usr/local/mariadb/lib/plugin --user=mysql --log-error=/data/mariadb/localhost.localdomain.err --pid-file=/data/mariadb/localhost.localdomain.pid --socket=/tmp/mysql.sock --port=3306
  root       2442  0.0  0.0 112668   964 pts/0    S+   00:21   0:00 grep --color=auto mysql
  
  [root@localhost mariadb]# netstat -tlnp
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1328/sshd
  tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      2032/master
  tcp6       0      0 :::3306                 :::*                    LISTEN      2404/mysqld
  ```

---

# Apache安装

Apache是一个基金会的名字，实际上httpd才是需要安装的软件包，早期其名字叫做apache。

## 下载httpd源码包及依赖包

- 安装httpd还需要使用`apr`和`apr-util`通用函数库，它们可以让httpd不关心底层操作系统平台，可以很方便的移植，如从linux移植到windows。

- 下载httpd和apr，apr-util源码包：

  ```bash
  // httpd2.4.33源码包
  # wget http://mirrors.cnnic.cn/apache/httpd/httpd-2.4.33.tar.gz
  // apr1.6.3源码包
  # wget http://mirrors.cnnic.cn/apache/apr/apr-1.6.3.tar.gz
  // apr-util1.6.1源码包
  # wget http://mirrors.cnnic.cn/apache/apr/apr-util-1.6.1.tar.bz2
  ```

## 编译apr和apr-util

- 由于yum安装的apr和apr-util版本与httpd2.4不匹配，所以需要手动编译这两个库；

### 编译apr

  ```bash
# tar zxvf apr-1.6.3.tar.gz
# cd apr-1.6.3
# ./configure --prefix=/usr/local/apr
# make && make install
  ```

- 检查是否安装正确

  ```bash
  [root@localhost apr-1.6.3]# echo $?
  0
  ```

### 编译apr-util

- 编译apr-util时需要指定apr的安装位置：

  ```bash
  # tar jxvf apr-util-1.6.1.tar.bz2
  # cd apr-util-1.6.1
  # ./configure --prefix=/usr/local/apr-util --with-apr=/usr/local/apr
  # make && make install
  ```

- 如果执行make的过程中报错如下，则需要安装xml解析的`expat-devel `软件包：

  ```bash
  [root@localhost apr-util-1.6.1]# make && make install
  make[1]: 进入目录“/usr/local/src/apr-util-1.6.1”
  /bin/sh /usr/local/apr/build-1/libtool --silent --mode=compile gcc -g -O2 -pthread   -DHAVE_CONFIG_H  -DLINUX -D_REENTRANT -D_GNU_SOURCE   -I/usr/local/src/apr-util-1.6.1/include -I/usr/local/src/apr-util-1.6.1/include/private  -I/usr/local/apr/include/apr-1    -o xml/apr_xml.lo -c xml/apr_xml.c &&touch xml/apr_xml.lo
  xml/apr_xml.c:35:19: 致命错误：expat.h：没有那个文件或目录
   #include <expat.h>
                     ^
  编译中断。
  make[1]: *** [xml/apr_xml.lo] 错误 1
  make[1]: 离开目录“/usr/local/src/apr-util-1.6.1”
  make: *** [all-recursive] 错误 1
  ```

  

## 编译安装httpd

- httpd的编译参数如下：

  ```bash
  ./configure --prefix=/usr/local/apache2.4 \
  --with-apr=/usr/local/apr \
  --with-apr-util=/usr/local/apr-util \
  --enable-so \
  --enable-mods-shared=most
  ```

  - 其中，`enable-so`参数是让apache支持动态扩展和加载模块，如php模块，`enable-mods-shared=most`参数则是指定apache支持绝大多数的模块扩展。

- 编译时报错`pcre-config`，则需要安装`pcre-devel`软件包：

  ```bash
  checking for pcre-config... false
  configure: error: pcre-config for libpcre not found. PCRE is required and available from http://pcre.org/
  ```

- 安装：

  ```bash
  # make && make install
  ```

  - 安装时如果报错如下，则是因为编译`apr-util`时没有安装`expat-devel`，并且安装之后没有重新进行编译，如果编译出错，使用`make clean`清除之前的编译结果：

  ```bash
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_GetErrorCode'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_SetEntityDeclHandler'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_ParserCreate'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_SetCharacterDataHandler'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_ParserFree'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_SetUserData'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_StopParser'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_Parse'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_ErrorString'
  /usr/local/apr-util/lib/libaprutil-1.so: undefined reference to `XML_SetElementHandler'
  collect2: error: ld returned 1 exit status
  make[2]: *** [htpasswd] 错误 1
  make[2]: 离开目录“/usr/local/src/httpd-2.4.33/support”
  make[1]: *** [all-recursive] 错误 1
  make[1]: 离开目录“/usr/local/src/httpd-2.4.33/support”
  make: *** [all-recursive] 错误 1
  ```

- 安装完成后可以进入apache的安装目录，apache的执行文件为`apache/bin/httpd`:

  ```bash
  [root@localhost apache2.4]# ls
  bin    cgi-bin  error   icons    logs  manual
  build  conf     htdocs  include  man   modules
  
  [root@localhost apache2.4]# ls -l bin/httpd
  -rwxr-xr-x. 1 root root 2348432 5月  25 01:33 bin/httpd
  ```

- apache的安装目录下，conf目录为配置文件所在目录，htdocs则是存放默认访问页的目录，logs则是日志存放的目录，日志由错误日志、访问日志等，modules则是扩展模块的存放目录，每个模块都代表apache的一个功能，查看apache加载了哪些模块，可以使用如下命令：

  ```bash
  # /usr/local/apache2.4/bin/apachectl -M
  or
  # /usr/local/apache2.4/bin/httpd -M
  ```

  ```bash
  [root@localhost apache2.4]# ./bin/httpd -M
  AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain. Set the 'ServerName' directive globally to suppress this message
  Loaded Modules:
   core_module (static)
   so_module (static)
   http_module (static)
   mpm_event_module (static)
   authn_file_module (shared)
   authn_core_module (shared)
   authz_host_module (shared)
   authz_groupfile_module (shared)
   authz_user_module (shared)
   authz_core_module (shared)
   access_compat_module (shared)
   auth_basic_module (shared)
   reqtimeout_module (shared)
   filter_module (shared)
   mime_module (shared)
   log_config_module (shared)
   env_module (shared)
   headers_module (shared)
   setenvif_module (shared)
   version_module (shared)
   unixd_module (shared)
   status_module (shared)
   autoindex_module (shared)
   dir_module (shared)
   alias_module (shared)
  ```

  - 这里模块后面的`static`和`shared`，static表示模块被编译进了htppd脚本里，是一个整体；shared则表示扩展模块，模块在modules目录下，在需要时被调用。

- 启动apache，执行`bin/apachectl start`：

  ```bash
  [root@localhost apache2.4]# /usr/local/apache2.4/bin/apachectl start
  AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using localhost.localdomain. Set the 'ServerName' directive globally to suppress this message
  
  [root@localhost apache2.4]# ps aux | grep httpd
  root      76399  0.0  0.2  70952  2248 ?        Ss   01:47   0:00 /usr/local/apache2.4/bin/httpd -k start
  daemon    76400  0.0  0.4 359916  4264 ?        Sl   01:47   0:00 /usr/local/apache2.4/bin/httpd -k start
  daemon    76401  0.0  0.4 359916  4264 ?        Sl   01:47   0:00 /usr/local/apache2.4/bin/httpd -k start
  daemon    76402  0.0  0.4 359916  4264 ?        Sl   01:47   0:00 /usr/local/apache2.4/bin/httpd -k start
  root      76485  0.0  0.0 112724   976 pts/0    S+   01:47   0:00 grep --color=auto httpd
  
  [root@localhost apache2.4]# netstat -tlnp
  Active Internet connections (only servers)
  Proto Recv-Q Send-Q Local Address           Foreign Address         State     PID/Program name
  tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     1328/sshd
  tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     2032/master
  tcp6       0      0 :::80                   :::*                    LISTEN     76399/httpd
  ```

  - 默认监听80端口。

---

