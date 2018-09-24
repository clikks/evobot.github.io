---
title: 'Centos7系列:文件及目录管理相关命令(一)'
author: Evobot
abbrlink: 9e44b9f5
date: 2018-03-23 22:04:24
categories: Centos7
tags: [Linux, Centos]
image: 
photo:
---

本文主要介绍Centos7的目录结构，文件类型，以及一些基础命令，如`ls`、`alias`命令，来进一步了解Centos系统的基本操作。

![Linux目录结构](http://qiniu.evobot.cn/linux%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84.jpg)

<!-- more -->

---

# 系统目录结构

- Linux的系统目录组织方式是树形的，从根目录`/`开始，其下又有多个子目录：

  ```bash
  [root@evobot ~]# ls /
  bin   dev  home  lib64       media  opt   root  sbin  sys  usr
  boot  etc  lib   lost+found  mnt    proc  run   srv   tmp  var
  ```

- 我们的shell提示符是`[root@evobot ~]#`，其中**root**表示当前用户名，**evobot**表示主机名，**~**表示当前所在目录为家目录，root用户的家目录就是根目录下的`root`目录，后面的** \# **表示当前是root用户，如果是普通用户，则是**$**符号，普通用户的家目录都在根目录下的`home`目录下；

- 家目录主要是存贮与用户相关的配置文件；

- 我们可以使用`tree`命令直观的看出系统的目录结构，默认Centos并没有安装`tree`命令，使用`yum -y install tree`即可安装该命令，`tree`命令的参数`-L`可以指定显示的目录层数：

```bash
  [root@evobot ~]# tree -L 1 /
  /
  |-- bin -> usr/bin
  |-- boot
  |-- dev
  |-- etc
  |-- home
  |-- lib -> usr/lib
  |-- lib64 -> usr/lib64
  |-- lost+found
  |-- media
  |-- mnt
  |-- opt
  |-- proc
  |-- root
  |-- run
  |-- sbin -> usr/sbin
  |-- srv
  |-- sys
  |-- tmp
  |-- usr
  `-- var

  20 directories, 0 files
```

- 系统目录作用说明
<style>
table th:first-of-type {
    width: 85px;
}
table th {
    text-align: center;
}
</style>


|    目录名     | 说明                                                         |
| :-----------: | ----------------------------------------------------------- |
|     /bin      | bin目录主要存放的是系统的命令，我们所使用的命令如`ls`、`yum`等，都存放于bin目录下，bin目录不止有`/bin`，还有`/usr/bin/`、`/usr/sbin/`、`/sbin/`也是同样的作用，区别是`sbin`目录下的命令是超级用户root使用的命令； |
|     /boot     | boot目录主要存放系统的启动文件，当我们启动系统时，进入的`grub`界面，就是存放在boot目录下； |
|     /dev      | dev目录是存放设备文件的目录，在Linux系统中，一切皆文件，所以即使是硬盘，光驱，在目录下呈现的也是一个文件； |
| /lib & /lib64 | lib及lib64是存放系统的库文件的目录，很多命令会依赖库文件，查看一个命令依赖哪些库，可以使用`ldd`命令：`ldd /bin/ls`; |
|     /proc     | 系统进程目录，系统的每个进程都会在这个目录下创建自己的进程文件和目录；在proc目录下存在以数字命名的目录，这就是进程的PID，在PID目录下可以看到`cwd`文件指向一个位置，这就是进程运行所在的目录； |
|     /run      | 进程运行存放临时文件的目录，在系统重启或关机后目录会清空；   |
|     /srv      | 存放服务产生的文件，默认为空；                               |
|     /sys      | 存放系统内核相关的文件，一般不会使用到这个目录；             |
|     /tmp      | 临时目录，任何用户都可以在其下创建文件；                     |
|     /usr      | 系统用户使用到的命令或文件、库文件的存放位置；一般安装的服务都会放在/usr/local目录下； |
|     /var      | 存放系统运行相关的文件，如日志/var/log；进程PID等。          |

# ls命令

## 命令语法

- ls命令主要用来显示目标列表，是使用率较高的命令，同时Centos系统默认ls命令的输出信息会进行彩色加亮显示，以区分不同了类型的文件。
- ls的语法是`ls (选项) (目录或文件)`；

## 常用选项

| 选项 | 作用                                                         |
| :--: | ------------------------------------------------------------ |
| `-l` | 输出详细信息，从左到右分别是文件类型、权限、硬链接数、所有者、组、文件大小、最后修改时间以及文件名 ； |
| `-i` | 显示文件的索引节点号，即`inode`号，`inode`记录文件存储在磁盘的所在块和区信息，如果两个文件inode号一样，则两个文件相同； |
| `-h` | 人性化显示文件大小，会在文件大小后显示单位，如`K`,`MB`;      |
| `-a` | 显示所有文件及目录，包括`.`开头的隐藏文件和目录，而`.`和`..`表示当前目录和上层目录，`.`所表示的当前目录的详细信息中的硬链接数，也表示当前目录下有多少个子目录； |
| `-t` | 以时间顺序排序显示，越晚的排在越上面；                       |
| `-d` | 只列出目录本身，不列出目录下的文件和子目录；                 |

- 系统中还存在着命令的别名，比如`ll`就等于`ls -l`，查看别名代表的详细命令，可以使用`which (命令)`来查看：

  ```bash
  [root@evobot ~]# ll
  total 4
  drwxr-xr-x 2 root root 4096 Mar 12 17:42 source
  [root@evobot ~]# which ll
  alias ll='ls -l --color=auto'	# --color=auto表示彩色高亮显示
  	/bin/ls
  ```

# 文件类型

- 在使用`ls -l`命令时，左边第一位表示文件的类型，文件类型有以下几种：

| 符号表示 | 代表类型                                         |
| :------: | ----------------------------------------------- |
|   `d`    | 表示目录；                                       |
|   `-`    | 表示普通文件，如文本文档，二进制文件；           |
|   `c`    | 表示字符串设备，如键盘，鼠标设备文件；           |
|   `l`    | 表示软连接文件，类似于windows的快捷方式；        |
|   `b`    | 块设备，如光驱，磁盘文件；                       |
|   `s`    | 表示socket文件，socket文件是进程用来通信的文件。 |

# 命令别名

## 查看命令别名

- 前面使用`which`查看命令别名所表示的实际命令，而命令别名是使用`alias`来定义的；

- 执行`alias`可以查看系统所有的命令别名和其所代表的实际命令：

  ```bash
  [root@evobot ~]# alias
  alias cp='cp -i'
  alias egrep='egrep --color=auto'
  alias fgrep='fgrep --color=auto'
  alias grep='grep --color=auto'
  alias l.='ls -d .* --color=auto'
  alias ll='ls -l --color=auto'
  alias ls='ls --color=auto'
  alias mv='mv -i'
  alias rm='rm -i'
  alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
  ```

- 可以看到`which`命令其实也是命令别名，实际上`which`是在系统的`PATH`环境变量所指向的目录中区查找我们要查看的命令，查看`PAHT`环境变量：

  ```bash
  [root@evobot ~]# echo $PATH
  /usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
  ```

## 设置别名

- 设置别名可以直接使用`alias (别名) (原命令 -选项/参数)`来设置：

  ```bash
  [root@evobot ~]# alias show='ls -lha'
  [root@evobot ~]# show
  total 52K
  dr-xr-x---.  6 root root 4.0K Mar 14 23:22 .
  dr-xr-xr-x. 18 root root 4.0K Mar 24 00:04 ..
  -rw-------   1 root root 3.7K Mar 24 00:04 .bash_history
  -rw-r--r--.  1 root root   18 Dec 29  2013 .bash_logout
  -rw-r--r--.  1 root root  176 Dec 29  2013 .bash_profile
  -rw-r--r--.  1 root root  176 Dec 29  2013 .bashrc
  drwxr-xr-x   3 root root 4.0K Mar  8 15:46 .cache
  drwxr-xr-x   3 root root 4.0K Mar  8 15:46 .config
  -rw-r--r--.  1 root root  100 Dec 29  2013 .cshrc
  drwxr-xr-x   2 root root 4.0K Mar 12 17:42 source
  drwx------   2 root root 4.0K Mar 12 14:25 .ssh
  -rw-r--r--.  1 root root  129 Dec 29  2013 .tcshrc
  -rw-------   1 root root 3.9K Mar 14 23:22 .viminfo
  ```

- 想要取消命令别名，则使用`unalias (别名)`即可，`unalias`有一个`-a`参数，可以取消系统所有的命令别名：

  ```bash
  [root@evobot ~]# unalias show
  [root@evobot ~]# show
  -bash: show: command not found
  ```

- 这样直接使用命令设置命令的别名，只会作用于当前登陆的操作，一旦退出登陆或重启，使用`alias`命令直接设置的别名也会被清除，如果需要每次登陆都能使用别名，则需要将相应的`alias`命令写入`/etc/bashrc`文件中，如果限定指定用户的别名，则写入用户家目录下的`.bashrc`文件中。

---

