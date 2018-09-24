---
title: 'Centos7系列:文件和目录权限(一)'
author: Evobot
abbrlink: 7af3ca57
date: 2018-03-28 21:00:04
categories: Centos7
tags: [Linux, Centos]
image:
---



介绍文件和目录权限的概念，如何更改文件及目录的所有者和所属组的命令**chown**、介绍权限掩码**umask**，以及隐藏权限操作命令**lsattr**和**chattr**命令。

<!-- more -->

---

# 文件和目录权限

## 文件权限概念

- 使用`ls -l`命令时，可以看到文件和目录的权限，权限一共有9位，分成3段，例如`rw-r--r--`，其中每一段的3位权限位分别表示**可读**、**可写**、**可执行**，使用`rwx`代表三种权限：

  ```bash
  [root@evobot source]# ls -l
  总用量 256
  -rw-r--r-- 1 root root 245760 3月  12 17:41 go1.9.2.linux-amd64.zip
  -rw-r--r-- 1 root root   9051 3月  27 23:40 test.txt
  ```

- 第一段权限位表示文件所有者的权限，第二段代表文件所属组的权限，第三段表示除了所有者和所属组用户之外的其他用户权限，而文件的所有者和所属组则都是root：

  ![文件权限](http://qiniu.evobot.cn/15222440423936fdqcu2t.png?imageslim)

- 文件的权限`rwx`也可以使用数字表示，其中`r=4`，`w=2`，`x=1`，所以`rwx`整段权限为7，例如`rw-r--r--`权限，`rw-`是**6**，`r--`是**4**，所以整个权限等于**644**。

## 更改权限命令chmod

- `chmod 755 /root/.ssh`，使用`chmod`加数字权限的方式来更改文件权限，或者`chmod +[rwx]`来单独增加权限，`chmod -[rwx]`来单独减少权限；

- `chmod -R`，可以递归更改目录权限，使目录内的文件权限与目录相同；
- chmod更改权限还可以写为`chmod u=rwx,g=r,o=r test.txt`的形式；
- `chmod [ugoa][+-][rwx] test.txt`,其中"u","g","o"分别表示所有者，所属组，其他人，"a"表示所有，即[ugo]的权限均增加或减少。例如`chmod a+x test.txt`给所有者、所属组、其他人增加"x"权限,`chmod u+w test.txt`给所有者增加"w"权限。

```bash
[root@localhost ~]# chmod a+x test.txt 
[root@localhost ~]# ll
总用量 4
drwxr-xr-x. 3 root root   15 10月 20 22:55 1
-rw-------. 1 root root 1422 8月  19 07:25 anaconda-ks.cfg
-rwxr-xr-x. 1 root root    0 10月 23 22:04 test.txt
```

```bash
[root@localhost ~]# chmod g+w test.txt 
[root@localhost ~]# ll
总用量 4
drwxr-xr-x. 3 root root   15 10月 20 22:55 1
-rw-------. 1 root root 1422 8月  19 07:25 anaconda-ks.cfg
-rwxrwxr-x. 1 root root    0 10月 23 22:04 test.txt
```
##  更改所有者、所属组

### 更改所有者

- `chown`命令可以更改文件或目录的所有者，命令格式为`chown (username) (filename)`：

```bash
[root@evobot source]# ls -l
总用量 256
-rw-r--r-- 1 root root 245760 3月  12 17:41 go1.9.2.linux-amd64.zip
-rw-r--r-- 1 root root   9051 3月  27 23:40 test.txt
[root@evobot source]# chown lux test.txt 
[root@evobot source]# ls -l test.txt 
-rw-r--r-- 1 lux root 9051 3月  27 23:40 test.txt
```

###  更改所属组

- `chgrp`命令用来更改文件或目录的所属组：

```bash
[root@evobot source]# ls -l test.txt 
-rw-r--r-- 1 lux root 9051 3月  27 23:40 test.txt
[root@evobot source]# chgrp lux test.txt 
[root@evobot source]# ls -l test.txt 
-rw-r--r-- 1 lux lux 9051 3月  27 23:40 test.txt
```

- 更改文件或目录也可以使用`chown`命令，只需要使用`:`分隔所有者和所属组，即可对文件更改所有者和所属组，如果只想更改所属组，在`:`前面的所有者留空即可：

```bash
[root@evobot source]# ls -l test.txt 
-rw-r--r-- 1 lux lux 9051 3月  27 23:40 test.txt
[root@evobot source]# chown root:root test.txt 
[root@evobot source]# ls -l test.txt 
-rw-r--r-- 1 root root 9051 3月  27 23:40 test.txt
[root@evobot source]# chown :lux test.txt 
[root@evobot source]# ls -l test.txt 
-rw-r--r-- 1 root lux 9051 3月  27 23:40 test.txt
```

- 使用`.`分隔所有者和所属组同样也可以对文件的所有者和所属组进行更改，如`chown .evobot secure.log`：

```bash
[root@evobot ~]# ll
总用量 19148
-rw------- 1 root root     1179 4月  27 00:02 secure.log

[root@evobot ~]# chown .evobot secure.log

[root@evobot ~]# ll
总用量 19148
-rw------- 1 root evobot     1179 4月  27 00:02 secure.log
```

- `chown`同样具有`-R`选项，可以在更改目录的所有者或所属组时，同时更改目录下的文件的所有者和所属组：

```bash
[root@evobot ~]# ls -l source/
总用量 256
-rw-r--r-- 1 root root 245760 3月  12 17:41 go1.9.2.linux-amd64.zip
-rw-r--r-- 1 root lux    9051 3月  27 23:40 test.txt
[root@evobot ~]# chown -R lux source/
[root@evobot ~]# ls -l source/
总用量 256
-rw-r--r-- 1 lux root 245760 3月  12 17:41 go1.9.2.linux-amd64.zip
-rw-r--r-- 1 lux lux    9051 3月  27 23:40 test.txt
```

# 权限掩码umask

## umask概念

- 在新建一个文件时，默认会给文件分配一个权限，这个权限其实就是由`umask`控制的：

```bash
[root@evobot ~]# mkdir evobot
[root@evobot ~]# ls -ld evobot/
drwxr-xr-x 2 root root 4096 3月  28 22:04 evobot/
[root@evobot ~]# umask
0022
```

- 可以看到`umask`的值时`0022`，需要更改`umask`的值，直接使用`umask`加上要指定的权限掩码即可：

```bash
[root@evobot ~]# umask 0002
[root@evobot ~]# umask
0002
[root@evobot ~]# mkdir evobot2
[root@evobot ~]# touch file.txt
[root@evobot ~]# ls -l
总用量 12
drwxr-xr-x 2 root root 4096 3月  28 22:04 evobot
drwxrwxr-x 2 root root 4096 3月  28 22:08 evobot2
-rw-rw-r-- 1 root root    0 3月  28 22:08 file.txt
```

- 从上面的示例中可以看到，目录的权限变成了775，文件的权限变成了664，这是因为Linux中目录必须有`x`权限，否则无法浏览目录；

## 计算umask

- 计算umask掩码所定义的文件和目录权限，是使用文件666，目录777权限去减去umask的权限，**注意并不是用数字权限减去umask，而是使用rwx形式相减！**

  1. 例如umask更改为002，则目录权限为`rwxrwxrwx - -------w- = rwxrwxr-x`:

  ```bash
  [root@localhost ~]# mkdir mark
  [root@localhost ~]# ll
  总用量 4
  -rw-------. 1 root root   1422 8月  19 07:25 anaconda-ks.cfg
  drwxr-xr-x. 2 lux  root     18 10月 23 23:28 lux
  drwxrwxr-x. 2 root root      6 10月 24 23:25 mark
  -rwxrwxr-x. 1 lux  clikks    0 10月 23 22:04 test.txt
  ```
  2. 文件的权限则为`rw-rw-rw- - -------w- = rw-rw-r--`:

  ```bash
  [root@localhost ~]# touch umask_file.txt
  [root@localhost ~]# ll umask_file.txt 
  -rw-rw-r--. 1 root root 0 10月 24 23:28 umask_file.txt
  ```
  3. 当用权限位减去umask时，若权限位为“-”，umask为"r"或"w"或"x"，则相减后依旧为"-"，例如当umask为003时，文件的权限为`rw-rw-rw- - -------wx = rw-rw-r--`,"-"减去"x"依旧为"-",目录权限为`rwxrwxrwx - -------wx = rwxrwxr--`:

  ```bash
  [root@localhost ~]# umask 003
  [root@localhost ~]# touch maskfile.txt
  [root@localhost ~]# mkdir mark2
  [root@localhost ~]# ll maskfile.txt 
  -rw-rw-r--. 1 root root 0 10月 24 23:32 maskfile.txt
  [root@localhost ~]# ls -ld mark2
  drwxrwxr--. 2 root root 6 10月 24 23:33 mark2
  ```


# 隐藏权限

## **chattr**增加隐藏权限

- `chattr`有两个常用选项，`a`和`i`，使用`+-`来给文件或者目录增减权限。

### 对文件附加权限

1.  `chattr +a filename`给文件增加`a`权限，则文件可读可追加内容，不可删除，不可移动、改名。
```bash
[root@localhost ~]# chattr +a files.ini 
[root@localhost ~]# lsattr files.ini 
-----a---------- files.ini
[root@localhost ~]# head -n2 /etc/passwd > files.ini 
-bash: files.ini: 不允许的操作
[root@localhost ~]# head -n2 /etc/passwd >> files.ini 
[root@localhost ~]# rm files.ini 
rm：是否删除普通文件 "files.ini"？y
rm: 无法删除"files.ini": 不允许的操作
[root@localhost ~]# mv files.ini xxx.ini
mv: 无法将"files.ini" 移动至"xxx.ini": 不允许的操作
[root@localhost ~]# cp files.ini xxx.ini
[root@localhost ~]# ls
anaconda-ks.cfg  files.ini  xxx.ini
```
2. `chattr +i filename`命令给文件增加`i`权限，则文件可读不可写，不可追加内容，不可删除，不可移动，也不能`touch`更改文件时间。
```bash
[root@localhost ~]# chattr +i files.ini 
[root@localhost ~]# lsattr files.ini 
----ia---------- files.ini
[root@localhost ~]# chattr -a files.ini 
[root@localhost ~]# lsattr files.ini 
----i----------- files.ini
[root@localhost ~]# !head
head -n2 /etc/passwd >> files.ini 
-bash: files.ini: 权限不够
[root@localhost ~]# head -n2 /etc/passwd > files.ini 
-bash: files.ini: 权限不够
[root@localhost ~]# rm files.ini 
rm：是否删除普通文件 "files.ini"？y
rm: 无法删除"files.ini": 不允许的操作
[root@localhost ~]# mv files.ini xxx.ini
mv：是否覆盖"xxx.ini"？ y
mv: 无法将"files.ini" 移动至"xxx.ini": 不允许的操作
[root@localhost ~]# lsattr 
---------------- ./anaconda-ks.cfg
----i----------- ./files.ini
---------------- ./xxx.ini
```
### 对目录附加权限

1. `chattr +a dirname`给目录增加`a`权限，则目录不可改名，不可删除，目录下可以增加文件和目录，并对文件内容进行增加删除，但是不能对目录下的文件删除和更名。同样的也不能对目录下的子目录删除，更名、移动。
```bash
[root@localhost ~]# tree dirname/
dirname/
├── files.txt
└── firstdir
1 directory, 1 file
[root@localhost ~]# chattr +a dirname/
[root@localhost ~]# lsattr dirname/
---------------- dirname/firstdir
---------------- dirname/files.txt
[root@localhost ~]# lsattr -d dirname/
-----a---------- dirname/
[root@localhost ~]# rm -r dirname/
rm：是否进入目录"dirname/"? y
rm：是否删除目录 "dirname/firstdir"？y
rm: 无法删除"dirname/firstdir": 不允许的操作
[root@localhost ~]# rm -r dirname/files.txt 
rm：是否删除普通空文件 "dirname/files.txt"？y     
rm: 无法删除"dirname/files.txt": 不允许的操作
[root@localhost ~]# mv dirname/ xxx
mv: 无法将"dirname/" 移动至"xxx": 不允许的操作
[root@localhost ~]# mkdir -p dirname/seconddir
[root@localhost ~]# tree dirname/
dirname/
├── files.txt
├── firstdir
└── seconddir
2 directories, 1 file
[root@localhost ~]# rm -r dirname/seconddir/
rm：是否删除目录 "dirname/seconddir/"？y
rm: 无法删除"dirname/seconddir/": 不允许的操作
[root@localhost ~]# head -n2 /etc/passwd > dirname/files.txt 
[root@localhost ~]# head -n2 /etc/passwd >> dirname/files.txt 
[root@localhost ~]# cat dirname/files.txt 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
[root@localhost ~]# mv dirname/files.txt dirname/xxx.ini
mv: 无法将"dirname/files.txt" 移动至"dirname/xxx.ini": 不允许的操作
[root@localhost ~]# mv dirname/firstdir/ dirname/third
mv: 无法将"dirname/firstdir/" 移动至"dirname/third": 不允许的操作
[root@localhost ~]# rm -r dirname/firstdir/ 
rm：是否删除目录 "dirname/firstdir/"？y
rm: 无法删除"dirname/firstdir/": 不允许的操作
```
2. `chattr +i dirname`给目录增加`i`权限，则目录无法删除、更名、增加文件和子目录，目录下的文件及子目录无法删除及更名，文件不能增加和删除内容。
```bash
[root@localhost ~]# chattr +i dirname/
[root@localhost ~]# head -n2 >> dirname/files.ini 
-bash: dirname/files.ini: 权限不够
[root@localhost ~]# head -n2 > dirname/files.ini 
-bash: dirname/files.ini: 权限不够
[root@localhost ~]# mkdir -p dirname/second
mkdir: 无法创建目录"dirname/second": 权限不够
[root@localhost ~]# touch dirname/file2.txt
touch: 无法创建"dirname/file2.txt": 权限不够
```

- 删除文件的隐藏权限，使用`chattr -a filename`或者`chattr -i filename`命令。

## **lsattr**查看隐藏权限
- `lsattr`命令用来查看文件的隐藏权限，常用选项有`-d`查看目录权限， `-R`查看目录及目录下所有子目录及文件权限，`-a`查看目录内所有文件权限包括隐藏文件。

```bash
[root@localhost ~]# lsattr -d dirname/
-----a---------- dirname/
[root@localhost ~]# lsattr -R dirname/
---------------- dirname/firstdir

dirname/firstdir:
---------------- dirname/firstdir/secdir

dirname/firstdir/secdir:

---------------- dirname/firstdir/file1.txt

---------------- dirname/files.txt
[root@localhost ~]# touch dirname/.file2.txt
[root@localhost ~]# lsattr -a dirname/
-----a---------- dirname/.
---------------- dirname/..
---------------- dirname/firstdir
---------------- dirname/files.txt
---------------- dirname/.file2.txt
[root@localhost ~]# lsattr  dirname/
---------------- dirname/firstdir
---------------- dirname/files.txt
[root@localhost ~]# lsattr -Ra dirname/
-----a---------- dirname/.
---------------- dirname/..
---------------- dirname/firstdir

dirname/firstdir:
---------------- dirname/firstdir/.
-----a---------- dirname/firstdir/..
---------------- dirname/firstdir/secdir

dirname/firstdir/secdir:
---------------- dirname/firstdir/secdir/.
---------------- dirname/firstdir/secdir/..

---------------- dirname/firstdir/file1.txt

---------------- dirname/files.txt
---------------- dirname/.file2.txt
```

---

