---
title: Linux正则表达式——awk命令
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: c7f852f3
date: 2018-04-27 22:17:13
image:
---

**awk**是一种编程语言，用于在linux/unix下对文本和数据进行处理。数据可以来自标准输入(stdin)、一个或多个文件，或其它命令的输出。它支持用户自定义函数和动态正则表达式等先进功能，是linux/unix下的一个强大编程工具。它在命令行中使用，但更多是作为脚本来使用。awk有很多内建的功能，比如数组、函数等，这是它和C语言的相同之处，灵活性是awk最大的优势。

<!--more-->

---

#  awk命令用法

##  分段打印

- `awk`命令相比`sed`，可以实现分段匹配，使用`-F`制定分隔符，命令格式为`awk -F ':' '{print $1}' [file]`，其中`$1`表示分割出的第一段，`{}`内就是所执行的操作,同样的`awk`也不会对原文件进行修改;

  ```bash
  [root@evobot ~]# awk -F ':' '{print $1}' passwd 
  root
  bin
  daemon
  adm
  lp
  sync
  ```

- `awk '{print $0}' [file]`可以打印出文件所有内容，在指定分隔符时，使用`$0`效果相同：

  ```bash
  [root@evobot ~]# head -n 3 passwd | awk -F ':' '{print $0}' 
  root:x:0:0:root:/root:/bin/BASH
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  [root@evobot ~]# head -n 3 passwd | awk '{print $0}' 
  root:x:0:0:root:/root:/bin/BASH
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  ```

- `awk`未指定分隔符时,默认以空格或空白字符为分隔符:

  ```bash
  [root@evobot ~]# cat 1.txt 
  zx cc vv
  sai amsomd sad
  [root@evobot ~]# awk '{print $1,$3}' 1.txt 
  zx vv
  sai sad
  ```

- 从上面的命令可以看出,打印多个分段,使用`,`分割指定的段即可;

- 打印多个分段时,也可以指定输出分段之间的分隔符,只需要将`,`改为需要指定的符号并用双引号引起来即可:

  ```bash
  [root@evobot ~]# awk '{print $1"#"$3}' 1.txt 
  zx#vv
  sai#sad
  [root@evobot ~]# awk '{print $1"$"$3}' 1.txt 
  zx$vv
  sai$sad
  ```

##  匹配用法

- `awk`也可以实现匹配功能,命令为`awk '/pattern/' [file]`:

  ```bash
  [root@evobot ~]# head -n 3 passwd | awk '/login/'
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  ```

- 匹配时可以在指定的段内匹配,如指定`$1`匹配,那么即使后面的分段有能匹配上关键字的字符串,`awk`也不会匹配,命令用法为`awk -F ':' '$1 ~ /pattern/' [file]`,这里的`~`表示匹配的意思,匹配时可以使用正则表达式:

  ```bash
  [root@evobot ~]# head -n 3 passwd 
  root:x:0:0:root:/root:/bin/BASH
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin

  [root@evobot ~]# head -n 3 passwd | awk -F ':' '$1 ~ /bin/'
  bin:x:1:1:bin:/bin:/sbin/nologin

  [root@evobot ~]# head -n 3 passwd | awk -F ':' '$1 ~ /oo+/'
  root:x:0:0:root:/root:/bin/BASH
  ```

- `awk`可以进行多个条件的匹配:

  ```bash
  [root@evobot ~]# awk -F ':' '/root/ {print $1, $3} /evobot/ {print $3, $4}' passwd 
  root 0
  1002 1002
  [root@evobot ~]# grep -E 'root|evobot' passwd 
  root:x:0:0:root:/root:/bin/BASH
  evobot:x:1002:1002::/home/evobot:/bin/bash
  ```

##  数学运算

- `awk`匹配还可以进行数学运算如相等`==`,比较`>=`、`<=`和不等`!=`等条件进行匹配:

  ```bash
  [root@evobot ~]# awk -F ':' '$3==0 {print $1}' passwd 
  root

  [root@evobot ~]# awk -F ':' '$3>=1000 {print $1}' passwd 
  centos
  lux
  evobot

  [root@evobot ~]# awk -F ':' '$7!="/sbin/nologin" {print $0}' passwd 
  root:x:0:0:root:/root:/bin/BASH
  sync:x:5:0:BUS:/sbin:/bin/sync
  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
  halt:x:7:0:halt:/sbin:/sbin/halt
  games:x:12:100:games:/usr/games:/sbin/noooologin
  syslog:x:996:994::/home/syslog:/bin/false
  centos:x:1000:1000:Cloud User:/home/centos:/bin/BASH
  lux:x:1001:1001::/home/lux:/bin/bash
  evobot:x:1002:1002::/home/evobot:/bin/bash
  ```

- 如果对数字使用双引号`""`,`awk`会将其认为是一个字符串,在比较时会以ASCII码进行比较:

  ```bash
  [root@evobot ~]# awk -F ':' '$3>="1000" {print $0}' passwd 
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  sync:x:5:0:BUS:/sbin:/bin/sync
  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
  halt:x:7:0:halt:/sbin:/sbin/halt
  mail:x:8:12:mail:/var/spoool/mail:/sbin/nologin
  ```

- 除了数字或字符串比较之外，`awk`还可以对两个字段进行比较，如`$3<$4`这样的比较：

  ```bash
  [root@evobot ~]# awk -F ':' '$3<$4 {print $0}' passwd 
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  mail:x:8:12:mail:/var/spoool/mail:/sbin/nologin
  games:x:12:100:games:/usr/games:/sbin/noooologin
  ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin

  [root@evobot ~]# awk -F ':' '$3==$4 {print $0}' passwd 
  root:x:0:0:root:/root:/bin/BASH
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  nobody:x:99:99:Nobody:/:/sbin/nologin
  systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
  dbus:x:81:81:System message bus:/:/sbin/nologin
  centos:x:1000:1000:Cloud User:/home/centos:/bin/BASH
  lux:x:1001:1001::/home/lux:/bin/bash
  evobot:x:1002:1002::/home/evobot:/bin/bash

  [root@evobot ~]# awk -F ':' '$3>$4 {print $0}' passwd 
  sync:x:5:0:BUS:/sbin:/bin/sync
  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
  halt:x:7:0:halt:/sbin:/sbin/halt
  operator:x:11:0:operator:/rooooot:/sbin/nologin
  polkitd:x:999:997:User for polkitd:/:/sbin/nologin
  syslog:x:996:994::/home/syslog:/bin/false
  nginx:x:995:993:Nginx web server:/var/lib/nginx:/sbin/nologin
  ```

- 比较条件还可以多个一起使用，如`$3>"5" && $3<"7"`：

  ```bash
  [root@evobot ~]# awk -F ':' '$3>"5" && $3<"7"' passwd # 以ASCII码比较
  daemon:x:59:2:daemon:/sbin:/sbin/nologin
  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown

  [root@evobot ~]# awk -F ':' '$3>5 && $3<7' passwd	# 以数字大小比较 
  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown

  ```

- 条件或比较，使用`$3>1000 || $7=="/sbin/nologin"`:

  ```bash
  [root@evobot ~]# awk -F ':' '$3>1000 || $7=="/bin/BASH"' passwd 
  root:x:0:0:root:/root:/bin/BASH
  centos:x:1000:1000:Cloud User:/home/centos:/bin/BASH
  lux:x:1001:1001::/home/lux:/bin/bash
  evobot:x:1002:1002::/home/evobot:/bin/bash

  [root@evobot ~]# awk -F ':' '$3>1000 || $7 ~ /BASH/' passwd 
  root:x:0:0:root:/root:/bin/BASH
  centos:x:1000:1000:Cloud User:/home/centos:/bin/BASH
  lux:x:1001:1001::/home/lux:/bin/bash
  evobot:x:1002:1002::/home/evobot:/bin/bash
  ```

##  指定输出定界符

- `awk`不仅可以指定处理文档时的定界符，也可以指定输出时打印出来的定界符，命令格式为`awk -F ':' '{OFS="#"} $3>1000 || $7 ~/BASH/ {print $1,$3,$7} [file]'`:

  ```bash
  [root@evobot ~]# awk -F ':' '{OFS="#"} $3>1000 || $7 ~ /BASH/ {print $1,$3,$7}' passwd 
  root#0#/bin/BASH
  centos#1000#/bin/BASH
  lux#1001#/bin/bash
  evobot#1002#/bin/bash
  ```

- 匹配条件可以和`print`放在一个`{}`里，组成一个操作命令：

  ```bash
  [root@evobot ~]# awk -F ':' '{OFS="#"} {if ($3>1000) {print $1,$3,$7}}' passwd 
  lux#1001#/bin/bash
  evobot#1002#/bin/bash
  ```

##  内置变量

- 上面的指定`OFS`其实也是`awk`中的内置变量，常用的内置变量还有`NR`表示行、`NF`表示段：

  ```bash
  [root@evobot ~]# head -n 3 passwd | awk -F ':' '{print NR":"$0}'	#打印行号
  1:root:x:0:0:root:/root:/bin/BASH
  2:bin:x:1:1:bin:/bin:/sbin/nologin
  3:daemon:x:59:2:daemon:/sbin:/sbin/nologin

  [root@evobot ~]# head -n3 passwd | awk -F ':' '{print NF":"$0}'	#打印每行分成几段
  7:root:x:0:0:root:/root:/bin/BASH
  7:bin:x:1:1:bin:/bin:/sbin/nologin
  7:daemon:x:59:2:daemon:/sbin:/sbin/nologin
  ```

- `NR`和`NF`还可以作为判断条件，例如打印前十行，在`sed`中命令为`sed -n '1,10'p [file]`,使用`awk`的命令为`awk -F ':' 'NR<=10' [file]`:

  ```bash
  [root@evobot ~]# awk -F ':' 'NR<=10' passwd 
  root:x:0:0:root:/root:/bin/BASH
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:59:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  sync:x:5:0:BUS:/sbin:/bin/sync
  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
  halt:x:7:0:halt:/sbin:/sbin/halt
  mail:x:8:12:mail:/var/spoool/mail:/sbin/nologin
  operator:x:11:0:operator:/rooooot:/sbin/nologin
  ```

  ```bash
  # 打印前十行中第一段为root或者sync的行
  [root@evobot ~]# awk -F ':' 'NR<=10 && $1 ~/root|sync/' passwd 
  root:x:0:0:root:/root:/bin/BASH
  sync:x:5:0:BUS:/sbin:/bin/sync
  ```

- `NF`作为条件可以对段数进行判断，如打印分段数为6的行：

  ```bash
  [root@evobot ~]# awk -F ':' 'NF==6 {print $0}' passwd 
  adm:3:4:adm:/var/adm:/sbin/nologin
  ```

- 当使用`$NF`和`$NR`，这两个变量的意义不同，如`awk -F ':' '{print $NR":"$NF}' [file]`,这条命令会在每读取一行，输出`NR`对应的`$行号:$段数`，由于`NF`是固定的数字，所以`$NR`超过段数时，输出为空：

  ```bash
  [root@evobot ~]# awk -F ':' '{print $NR":"$NF'} passwd 
  root:/bin/BASH
  x:/sbin/nologin
  59:/sbin/nologin
  adm:/sbin/nologin
  lp:/sbin/nologin
  /sbin:/bin/sync
  /sbin/shutdown:/sbin/shutdown
  :/sbin/halt
  :/sbin/nologin
  :/sbin/nologin
  :/sbin/noooologin
  ```

##  赋值

- 使用`=`表示给变量赋值，如在`awk`中使用`$1="root"`，则所有的输出行的`$1`都变成了`root`：

  ```bash
  [root@evobot ~]# head -n 3 passwd | awk -F ':' '$1="awk"'
  awk x 0 0 root /root /bin/BASH
  awk x 1 1 bin /bin /sbin/nologin
  awk x 59 2 daemon /sbin /sbin/nologin

  [root@evobot ~]# head -n3 passwd | awk -F ':' '{OFS=":"} $1="root"'
  root:x:0:0:root:/root:/bin/BASH
  root:x:1:1:bin:/bin:/sbin/nologin
  root:x:59:2:daemon:/sbin:/sbin/nologin
  ```

## END用法

- 求和整个文件的一列数据并打印结果，需要使用`END`：

  ```bash
  [root@evobot ~]# awk -F ':' '{(sum=sum+$3)}; END {print sum}' passwd 
  8969
  ```

  这其中`{(sum=sum+$3)}`是我们自己定义的累加求和的公式，也可以使用`{(tot=tot+$3)}`,`tot`表示求和；

  `END`表示计算完成后最后执行的语句。

---