---
title: Shell基础(二)
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 49e2ed70
date: 2018-04-22 23:30:43
image:
---



本文继续介绍Shell的基础知识，涵盖管道符和作业控制的相关知识、Shell变量的相关知识以及系统环境变量配置文件的介绍。

<!--more-->

---

# Shell基础(二)

## 管道符和作业控制

- 管道符`|`表示把一个文件或命令的输出内容传递给后面的命令，使用形式如`cat 1.txt | wc -l`或`cat 1.txt | grep 'aaa'`;

```bash
[root@evobot tmp]# cat setRps.log | grep 'queues'
eth:eth0  queues:1
total_nic_queues:1  flow_entries:4096
```

- 上面的命令输出结果，就只包含管道符后面的`grep`筛选的内容。
- 作业控制表示控制程序运行状态，如暂停，继续运行，使用`ctrl+z`可以暂停当前任务，如暂停vim任务：

```bash
[root@evobot tmp]# vi

[1]+  已停止               vi
```

- 将已经暂停的任务调回前台，使用`fg`命令：

```bash
[root@evobot tmp]# fg
vi
```

- 当有多个任务被暂停时，使用命令`jobs`可以查看系统已经暂停的任务，显示出的任务会有一个序号，`fg`指定任务编号，即可将指定编号的任务调回前台：

```bash
[root@evobot tmp]# jobs
[1]-  已停止               vi aaa
[2]+  已停止               vi bbb
[root@evobot tmp]# fg 2
vi bbb
```

- 除了`fg`将任务调回前台，还可以使用`bg`命令让任务后台运行，例如`vmstat 1`命令，会持续输出系统状态，我们将其设置为后台运行：

```bash
[root@evobot tmp]# jobs
[1]+  已停止               vi aaa
[root@evobot tmp]# vmstat 1
procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 2  0      0 533176 225372 1026764    0    0     1    43  100  104  2  1 97  0  0
 1  0      0 533176 225372 1026764    0    0     0     0  111  116  0  0 100  0  0
^Z
[2]+  已停止               vmstat 1
[root@evobot tmp]# jobs
[1]-  已停止               vi aaa
[2]+  已停止               vmstat 1
[root@evobot tmp]# bg 2
[2]+ vmstat 1 &
[root@evobot tmp]#  0  0      0 532648 225376 1027132    0    0     0    20  509  585  0  0 100  0  0
 0  0      0 532648 225376 1027132    0    0     0    20  112  123  0  0 99  1  0
 0  0      0 532648 225376 1027132    0    0     0     0   93  111  0  1 99  0  0
jobs
[1]+  已停止               vi aaa
[2]-  运行中               vmstat 1 &

```

- 虽然`vmstat 1`命令在后台运行也会在前台打印，但是执行`jobs`命令可以看到任务状态是**运行中**，并且在命令的结尾有一个`&`符号，这表示任务在后台运行；
- 使用`fg`和`bg`命令，在不加任务编号的情况下，默认会调用最后一次暂停的任务；
- 在命令结尾加`&`，可以让命令在后台运行，并且也可以使用`jobs`查看任务：

```bash
[root@evobot tmp]# sleep 500
^Z
[2]+  已停止               sleep 500
[root@evobot tmp]# jobs
[1]-  已停止               vi aaa
[2]+  已停止               sleep 500
[root@evobot tmp]# sleep 1000 &
[3] 13618
[root@evobot tmp]# jobs
[1]-  已停止               vi aaa
[2]+  已停止               sleep 500
[3]   运行中               sleep 1000 &
```

- `jobs`命令只能查看当前tty的任务，另一个tty不能查看到其他tty的任务。

## shell变量

### 查看变量

- 如以前说过的`PATH`，就是系统的一个环境变量，查看系统所以的变量，可以使用`env`命令查看，一般系统变量以大写字符串定义：

  ```bash
  [root@evobot tmp]# env
  XDG_SESSION_ID=5806
  HOSTNAME=evobot
  SHELL=/bin/bash
  TERM=xterm-256color
  HISTSIZE=3000
  USER=root
  ```

- `set`命令也可以查看变量，并且不仅列出系统变量，还会输出用户定义的变量：

  ```bash
  [root@evobot tmp]# set
  ABRT_DEBUG_LOG=/dev/null
  BASH=/bin/bash
  BASHOPTS=checkwinsize:cmdhist:expand_aliases:extglob:extquote:force_fignore:histappend:interactive_comments:login_shell:progcomp:promptvars:sourcepath
  BASH_ALIASES=()
  BASH_ARGC=()
  BASH_ARGV=()
  BASH_CMDS=()
  BASH_COMPLETION_COMPAT_DIR=/etc/bash_completion.d
  BASH_LINENO=()
  BASH_REMATCH=()
  BASH_SOURCE=()
  BASH_VERSINFO=([0]="4" [1]="2" [2]="46" [3]="2" [4]="release" [5]="x86_64-redhat-linux-gnu")
  BASH_VERSION='4.2.46(2)-release'
  ```

### 自定义变量

- 自定义变量直接使用`=`复制即可，查看变量使用`echo $[变量名]`：

  ```bash
  [root@evobot tmp]# myvar=abc
  [root@evobot tmp]# echo $myvar
  abc
  [root@evobot tmp]# set | grep 'myvar'
  myvar=abc
  ```

- Shell自定义变量的命名规则必须由**字母、数字或下划线**组成，并且**首位不能为数字**；变量的值存在特殊符号时，值需要使用单引号括起来，单引号可以将特殊字符脱义，而双引号则不行：

  ```bash
  [root@evobot tmp]# 1a=111
  -bash: 1a=111: 未找到命令
  [root@evobot tmp]# a1=111
  [root@evobot tmp]# echo $a1
  111
  [root@evobot tmp]# a_2=zxcv
  [root@evobot tmp]# echo $a_2
  zxcv

  [root@evobot tmp]# var1=a b c
  -bash: b: 未找到命令
  [root@evobot tmp]# var1='a b c'
  [root@evobot tmp]# echo $var1
  a b c
  [root@evobot tmp]# var2="a$bc"
  [root@evobot tmp]# echo $var2
  a
  [root@evobot tmp]# var2='a$bc'
  [root@evobot tmp]# echo $var2
  a$bc
  ```

- 变量之间可以进行累加，如将两个变量的值进行拼接：

  ```bash
  [root@evobot tmp]# var3=1
  [root@evobot tmp]# var4=2
  [root@evobot tmp]# echo $var3$var4
  12
  [root@evobot tmp]# echo "a$var4"c
  a2c
  [root@evobot tmp]# echo "a$bc"	# $后面被识别为变量bc
  a
  ```

### 全局变量

- 使用命令`w`可以查看当前登录的客户端和用户，使用TTY表示，查看当前用户的TTY，可以查看变量`SSH_TTY`：

  ```bash
  [lux@evobot ~]$ w
   00:38:58 up 4 days, 3 min,  2 users,  load average: 0.00, 0.01, 0.05
  USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
  lux      pts/0    118.113.33.203   23:36    1:06   0.17s  0.00s sshd: lux [priv]    
  lux      pts/1    118.113.33.203   00:38    1.00s  0.03s  0.00s w

  [lux@evobot ~]$ echo $SSH_TTY
  /dev/pts/1
  ```

- 普通的自定义变量只能在当前bash中访问，例如在当前shell中新建`bash`,新的bash中就无法再访问父shell的自定义变量：

  ```bash
  [lux@evobot ~]$ echo $var1

  [lux@evobot ~]$ var1=abc
  [lux@evobot ~]$ echo $var1
  abc
  [lux@evobot ~]$ bash
  [lux@evobot ~]$ echo $var1

  [lux@evobot ~]$ 
  ```


- 定义全局变量使用`export [变量名=值]`，需要注意的是，全局变量只能在子shell中生效，而子shell中的全局变量无法在父shell中生效，也不会在其他的tty中生效：

  ```bash
  [lux@evobot ~]$ echo $SSH_TTY
  /dev/pts/2
  [lux@evobot ~]$ echo $var1

  [lux@evobot ~]$ 

  [lux@evobot ~]$ export var2=cdef
  [lux@evobot ~]$ echo $var2
  cdef
  [lux@evobot ~]$ exit
  exit
  [lux@evobot ~]$ echo $var2

  [lux@evobot ~]$ 
  ```

- 使用`unset`命令可以取消变量，命令用法为`unset [变量名]`:

  ```bash
  [lux@evobot ~]$ echo $var1
  centosvar
  [lux@evobot ~]$ unset var1
  [lux@evobot ~]$ echo $var1

  ```

---

## 环境变量配置文件

- Linux中的环境变量配置文件分为系统全局配置和用户配置，常见的配置文件有以下几个：

  - 系统层次环境变量配置文件：

    >  **/etc/profile** 用户环境变量,登录、交互才执行;
    >
    >  **/etc/bashrc** 用户不用登录，执行shell就生效；

  - 用户层次环境变量配置文件，每个用户的家目录下都会有以下文件：

    > **~/.bashrc**
    >
    > **~/.bash_profile**
    >
    > **~/.bash_history**
    >
    > **~/bash_logout**

  - 其中`profile`命名的文件是与登录相关的配置，`bashrc`命名的文件，则是与系统或用户执行命令和脚本相关的配置；


### bashrc和bash_profile的区别

- 当直接在机器login界面登陆、使用ssh登陆或者su切换用户登陆时，`.bash_profile` 会被调用来初始化shell环境


- 查看`~/.bash_profile`的文件内容：

  ```bash
  # .bash_profile

  # Get the aliases and functions
  if [ -f ~/.bashrc ]; then
          . ~/.bashrc		# 这里的.符号与source命令作用相同,.bash_profile会自动加载~/.bashrc;
  fi

  # User specific environment and startup programs

  PATH=$PATH:$HOME/bin

  export PATH
  ```

- 当不登陆系统而使用ssh直接在远端执行命令，.bashrc 会被调用，已经登陆系统后，每打开一个新的Terminal时，.bashrc 都会被再次调用，在`~/.bashrc`的文件内容中，又会自动加载`/etc/bashrc`：

  ```bash
  # .bashrc

  # User specific aliases and functions

  alias rm='rm -i'
  alias cp='cp -i'
  alias mv='mv -i'

  # Source global definitions
  if [ -f /etc/bashrc ]; then
          . /etc/bashrc
  fi
  ```

- 所以配置环境变量，最好是卸载`.bashrc`文件中，因为不论是登陆还是不登陆，该文件都会被调用。

### 其他环境变量文件

- `～/.bash_logout`文件用来定义用户退出时系统所做的操作，如用户退出时清除命令历史，那么将相关命令写入文件即可；

- `PS1`变量使用来定义用户登录后的提示符格式的，如`[root@evobot ~]#`，这样的格式就是在`PS1`变量中定义的，`PS1`变量的定义在`/etc/bashrc`中：

  ```bash
  [root@evobot ~]# echo $PS1
  [\u@\h \W]\$
  # 这里\u表示用户名,\h表示hostname，\W表示当前目录

  [root@evobot ~]# cat /etc/bashrc | grep '&& PS1='
    [ "$PS1" = "\\s-\\v\\\$ " ] && PS1="[\u@\h \W]\\$ "
  ```

- 如果将`\W`改为`\w`，则当前目录将显示绝对路径，其他能够更改的选项如下表格所示：

<style>
table th:first-of-type {
    width: 130px;
    text-align: center;
}
</style>

| 设置选项 |             作用             |
| :--: | :------------------------: |
| `\d` | 表示日期，格式为weekday month date |
| `\H` |           完整的主机名           |
| `\h` |          主机的第一个名字          |
| `\t` |     24小时格式(HH:MM:SS)时间     |
| `\T` |           12小时格式           |
| `\A` |    24小时时间格式(HH:MM)，不包括秒    |
| `\u` |           当前用户名            |
| `\v` |         BASH的版本信息          |
| `\w` |         工作目录的绝对路径          |
| `\w` |         当前工作目录的目录名         |
| `\#` |           第几个命令            |
| `\$` |  提示字符，root用户为`#`，普通用户为`$`  |

- 对于`PS1`，不仅可以设置格式，也可以设置颜色。
- 除了`PS1`之外，还有`PS2`变量，`PS2`变量用来定义进入shell下的程序提供的终端时显示的格式，默认`PS2`的变量值为`>`。

---

## 简单操作记录审计

- 在需要针对用户操作进行历史记录以便出现问题时查找责任人时，由于`history`命令用户能够自行删除，所以可以通过一些配置来记录用户的操作记录；

- 首先创建所需要的目录：

  ```bash
  [root@evobot ~]# mkdir -p /usr/local/records/
  [root@evobot ~]# chmod 777 /usr/local/records/
  [root@evobot ~]# chmod +t /usr/local/records/
  ```

- 在`/etc/profile`中添加如下的代码：

  ```bash
  if [ ! -d /usr/local/records/${LOGNAME} ];then
      mkdir -p /usr/local/records/${LOGNAME}
      chmod 300 /usr/local/records/${LOGNAME}
  fi

  export HISTORY_FILE="/usr/local/records/${LOGNAME}/bash_history"
  export PROMPT_COMMAND='{ date +"%Y-%m-%d %T ##### $(who am i |awk "{print \$1\" \"\$2\" \"\$5}") #### $(history 1 | { read x cmd; echo "$cmd";})";} >> $HISTORY_FILE'
  ```

- 这样在用户登录时，会在`/usr/local/records`目录下创建一个与用户名同名的目录，目录下的`bash_history`会将用户的历史命令记录下来，其中`PROMPT_COMMAND`变量是在显示`PS1`变量之前执行的，所以这里用户每执行一次命令再显示下一个`PS1`之前，都会执行`PROMPT_COMMAND`的定义的内容；

- 这里定义的`PROMPT_COMMAND`的内容就是历史记录的格式，首先输出日期和时间，然后截取用户的用户名，tty和IP地址，`history 1 | { read x cmd; echo "$cmd";}`表示取`history`命令的最后一个记录，`history`命令格式如下：

  ```bash
  [lux@evobot ~]$ history 1
    308  history 1
  ```

- `{ read x cmd; echo "$cmd"; }`则表示将`history 1`的输出分别赋值给`x`和`cmd`，然后打印`cmd`变量的值；

- 这样一个简单的用户历史命令审计系统就可以运行了，但是实际上用户仍然可以更改目录的权限，对命令历史进行更改。

---

