title: 'Centos7系列:文件及目录管理相关命令(三)'
author: Evobot
abbrlink: 8385c26e
categories: Centos7
tags:
  - Linux
  - Centos
date: 2018-03-27 22:29:06
image:
---
> 主要介绍Linux环境变量PATH，以及移动`mv`、复制`cp`命令的使用，另外则是查看文档操作相关的命令:`cat`、`more`、`less`和`head`。

<!-- more -->

# 环境变量PATH

## 查看PATH

- 使用`which`命令可以查看命令的别名，而`which`命令并不是在整个系统去查找命令，而是从环境变量PATH的目录中去查找，查看PATH包含的目录，使用`echo $PATH`：

  ```bash
  [root@evobot ~]# echo $PATH
  /usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
  ```

- 实际上在LInux中执行一个命令，是需要使用绝对路径才可以执行，但是有了PATH，我们执行命令时，系统会自动到PATH包含的目录下去查找调用。

- 如果将一个命令放到不包含在PATH变量中的目录下，那么只能使用绝对路径的方式去执行这个命令：

  ```bash
  [root@evobot ~]# which ls
  alias ls='ls --color=auto'
  	/bin/ls
  [root@evobot ~]# cp /bin/ls /tmp/ls2
  [root@evobot ~]# ls2
  -bash: ls2: 未找到命令
  [root@evobot ~]# /tmp/ls2
  source
  ```

## 增加PATH包含的目录

- 如果想将一个目录包含到PATH中，执行如下命令:

  ```bash
  [root@evobot ~]# PATH=$PATH:/tmp/
  [root@evobot ~]# echo $PATH
  /usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:/tmp/
  [root@evobot ~]# ls2
  source
  ```

- 上面的方式只会在当前登录用户的环境中生效，而要让新的环境变量永久生效，则需要将`PATH=$PATH:/tmp/`写入到`/etc/profile`文件末尾。

  ![增加PATH](http://qiniu.evobot.cn/1522162144804yntgfqme.png?imageslim)

# 复制命令 cp

- `cp`命令是LInux中用来复制文件和目录的命令，拷贝文件直接使用`cp (/文件路径/文件名) /目的路径/复制文件名`命令即可：

  ```bash
  [root@evobot ~]# cp source/go1.9.2.linux-amd64.tar.gz /tmp/test.tar.gz
  [root@evobot ~]# ls /tmp/test.tar.gz 
  /tmp/test.tar.gz
  ```

- 如果要复制目录，则使用`cp -r`选项，并且最好在目录路径后添加`/`表示目录：

  ```bash
  [root@evobot ~]# ls source/
  go1.9.2.linux-amd64.tar.gz  go1.9.2.linux-amd64.tar.gz.1
  [root@evobot ~]# cp -r source/ /tmp/source2/
  [root@evobot ~]# ls /tmp/source2/
  go1.9.2.linux-amd64.tar.gz  go1.9.2.linux-amd64.tar.gz.1
  ```

- 当使用`cp -r`复制一个目录到目标，但目标目录已经存在的情况下，`cp -r`命令会将源目录放到目标目录下，当目标目录下同样存在与源目录同名的目录时，则会询问用户是否要覆盖目录：

  ```bash
  [root@evobot ~]# ls /tmp/source2/
  go1.9.2.linux-amd64.tar.gz  go1.9.2.linux-amd64.tar.gz.1
  [root@evobot ~]# cp -r source/ /tmp/source2/
  [root@evobot ~]# tree /tmp/source2/
  /tmp/source2/
  |-- go1.9.2.linux-amd64.tar.gz
  |-- go1.9.2.linux-amd64.tar.gz.1
  `-- source
      |-- go1.9.2.linux-amd64.tar.gz
      `-- go1.9.2.linux-amd64.tar.gz.1

  1 directory, 4 files
  [root@evobot ~]# cp -r source/ /tmp/source2/
  cp：是否覆盖"/tmp/source2/source/go1.9.2.linux-amd64.tar.gz.1"？ y
  cp：是否覆盖"/tmp/source2/source/go1.9.2.linux-amd64.tar.gz"？ y
  [root@evobot ~]# tree /tmp/source2/
  /tmp/source2/
  |-- go1.9.2.linux-amd64.tar.gz
  |-- go1.9.2.linux-amd64.tar.gz.1
  `-- source
      |-- go1.9.2.linux-amd64.tar.gz
      `-- go1.9.2.linux-amd64.tar.gz.1

  1 directory, 4 files
  ```

# 移动命令mv

- `mv`命令用来移动文件和目录，如果`mv`的源文件路径与目的路径相同，则是重命名的作用：

  ```bash
  [root@evobot source]# ls
  go1.9.2.linux-amd64.tar.gz  go1.9.2.linux-amd64.tar.gz.1
  [root@evobot source]# mv go1.9.2.linux-amd64.tar.gz go1.9.2.linux-amd64.zip
  [root@evobot source]# ls
  go1.9.2.linux-amd64.tar.gz.1  go1.9.2.linux-amd64.zip
  [root@evobot source]# mv go1.9.2.linux-amd64.tar.gz.1 /tmp/
  [root@evobot source]# ls
  go1.9.2.linux-amd64.zip
  [root@evobot source]# ls /tmp/
  evobot1                       setRps.log
  go1.9.2.linux-amd64.tar.gz.1
  ```

- `mv`命令也可以在移动文件的同时，将文件重命名:

  ```bash
  [root@evobot source]# mv test1.txt /tmp/test2.txt
  [root@evobot source]# tree /tmp/
  /tmp/
  |-- net_affinity.log
  |-- setRps.log
  |-- systemd-private-e6b2bdd2f0d3443aa619bc608648720d-ntpd.service-F90V6C
  |   `-- tmp
  `-- test2.txt

  2 directories, 3 files
  ```

- 同样的，当目的路径下存在同名文件时，`mv`命令会询问是否覆盖，也可以使用`mv -f`选项，强制覆盖。

# 文档查看命令

## cat及tac命令

- `cat`命令用来查看文件内容，会将文件内容全部打印到屏幕上，使用`cat -a`选项则可以将所有字符打印出来，比如文件内容行尾结束符，而`cat -n`选项则可以打印行号；
- 而`tac`命令则与`cat`命令相反，会将文件内容倒序打印出来。

## more命令

- `more`命令不会将文件内容一次性打印出来，而是会将文件内容显示一屏，按空格键则会显示下一屏内容，知道文件内容显示完则自动退出，而显示上一屏内容，按ctrl+b组合键。

## less命令

- `less`命令的操作与`more`类似，但是支持方向键上下滚动，同时`ctrl+f`也可以向后翻页，当翻页到文件末尾时，并不会同时退出显示，而是需要按`q`键退出；
- 在`less`查看文件内容时，可以使用`/`后面跟需要查找的内容，来在文件内容中进行搜索，搜索出多个结果时，按`n`则向下查看下一个搜索结果，`N`向前查看搜索结果；
- 使用`?`加搜索内容，则是从后向前搜索，同时`n`和`N`的功能与`/`搜索相反。
- `less`命令还有一些快捷键，如`shift+g`跳转到末行，`g`跳转到首行。

## head和tail命令

- `head`命令默认查看文件内容前10行，而`tail`则是查看文件内容末尾10行；

- `-n`选项则可以为`head`和`tail`指定显示的行数：

  ```bash
  [root@evobot source]# head -n 2 test.txt 
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  [root@evobot source]# tail -n 3 test.txt 
  centos:x:1000:1000:Cloud User:/home/centos:/bin/bash
  lux:x:1001:1001::/home/lux:/bin/bash
  nginx:x:995:993:Nginx web server:/var/lib/nginx:/sbin/nologin
  ```

- `tail`的选项`-f`可以动态查看文件内容，当文件内容增加时，会打印到屏幕上。

---