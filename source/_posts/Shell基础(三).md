---
title: Shell基础(三)
author: Evobot
abbrlink: 3711cd5b
date: 2018-04-23 23:31:37
categories: Sehll
tags:
  - Shell
image:
---



本文继续介绍Linux Shell相关基础知识，主要介绍Shell的特殊符号，介绍几个常用的shell命令，如**cut**、**sort**、**wc**、**uniq**几个对文件内容进行操作和排序统计的命令，以及**tee**、**tr**、**split**命令等。

<!--more-->

---

# Shell基础(三)

## Shell的特殊符号

- shell中有一些特殊符号，常用的有以下几种：

   **\*** ： 代表零个或多个任意字符；

   **?** ： 代表任意一个字符；

   **#** ：表示注释字符，以#号开头的命令或代码都是不生效的；

   **\\** ： 表示脱义 ，将一些特殊符号的表达为普通字符或符号；

   **|** ： 管道符，将符号左边的内容传递给符号右边；

   **$** ： 变量前缀，`!$`组合正则中表示行尾；

   **;** ： 多条命令写到一行执行，使用`;`分隔；

   **&** ： 放在命令结尾，表示后台执行；

   **||**和**&&** ： 用于多个命令之间，`&&`表示前一条执行正确再执行下一条，否则不执行后一条，`||`表示前一条执行不成功再执行下一条命令；


## 文档内容处理命令

###  **cut**命令

- `cut`命令可以截取文件内容中的字符串，提取自己需要的内容；

- `cut`命令在使用时，需要指定截取内容的分隔符，使用`-d`选项指定分隔符将内容分段，然后使用`-f`选项指定需要的字符串段数，如截取1,2段，则命令为`-f 1,2`，也可以指定范围`-f 1-4`：

  ```bash
  [root@evobot lux]# cat /etc/passwd | head -n 2
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  [root@evobot lux]# cat /etc/passwd | head -n 2 | cut -d ':' -f 1,2
  root:x
  bin:x
  [root@evobot lux]# cat /etc/passwd | head -n 2 | cut -d ':' -f 1-4
  root:x:0:0
  bin:x:1:1
  ```

- `cut`命令的`-c`选项可以指定截取第几个字符，并且使用`-c`选项时，不能使用`-d`和`-f`指定分隔符和范围：

  ```bash
  [root@evobot lux]# cat /etc/passwd | head -n 2 | cut -c 1
  r
  b
  [root@evobot lux]# cat /etc/passwd | head -n 2 | cut -c 2
  o
  i
  ```

### **sort**命令

- `sort`命令用来对文件内容进行排序，可以以数字排序，或者反序排列，并且也可以指定分隔符；

- `sort`命令常与`uniq`命令一起使用，默认直接使用`sort`对文件内容排序，是以内容的首字符的`ASCII`码排序：

  ```bash
  [root@evobot lux]# cat /etc/passwd | head -n 5 | sort
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  root:x:0:0:root:/root:/bin/bash
  ```

- `sort`的`-n`选项可以以数字进行排序，当使用`-n`选项时，字母和特殊符号会被`sort`当做0进行处理，所以字母和符号会排在数字前面：

  ```bash
  [root@evobot ~]# cat test.txt 
  1111
  1211
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  @#$%^&
  [root@evobot ~]# sort -n test.txt 
  @#$%^&
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  root:x:0:0:root:/root:/bin/bash
  1111
  1211
  ```

- `-r`选项是反序排列：

  ```bash
  [root@evobot ~]# sort -n -r test.txt 
  1211
  1111
  root:x:0:0:root:/root:/bin/bash
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  bin:x:1:1:bin:/bin:/sbin/nologin
  @#$%^&
  [root@evobot ~]# sort -r test.txt 
  root:x:0:0:root:/root:/bin/bash
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  bin:x:1:1:bin:/bin:/sbin/nologin
  1211
  1111
  @#$%^&
  ```

- 同样的，`sort`命令也可以指定分隔符，选项为`-t [分隔符]`，然后使用`-k`选项指定以分隔出的第几段进行排序，格式为`-k n`或`-k n1,n2`：

  ```bash
  [root@evobot ~]# head -n 5 /etc/passwd | sort -t ':' -k 1
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  root:x:0:0:root:/root:/bin/bash

  [root@evobot ~]# head -n 5 /etc/passwd | sort -t ':' -k 3,4
  root:x:0:0:root:/root:/bin/bash
  bin:x:1:1:bin:/bin:/sbin/nologin
  daemon:x:2:2:daemon:/sbin:/sbin/nologin
  adm:x:3:4:adm:/var/adm:/sbin/nologin
  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
  ```

- 一般`sort`命令将结果输出到终端，将结果写入文件可以使用重定向的方式，但是如果将结果写入原文件，使用重定向的方式会导致原文件被清空，所以这时需要使用`sort -o`参数：

  ```bash
  [root@evobot ~]# cat test.txt 
  123
  223
  333
  124
  234

  [root@evobot ~]# sort -n test.txt 
  123
  124
  223
  234
  333

  [root@evobot ~]# sort -n test.txt > test.txt 
  [root@evobot ~]# cat test.txt 
  [root@evobot ~]# sort -n test.txt -o test.txt 
  [root@evobot ~]# cat test.txt 
  123
  124
  223
  234
  333
  ```



### **wc**命令

- `wc`命令用来对文件内容的行数或字数，常用选项为`wc -l [filename]`统计行数和`wc -m [filename]` 统计字数：

  ```bash
  [root@evobot ~]# cat -n test.txt 
       1	1111
       2	1211
       3	root:x:0:0:root:/root:/bin/bash
       4	bin:x:1:1:bin:/bin:/sbin/nologin
       5	daemon:x:2:2:daemon:/sbin:/sbin/nologin
       6	@#$%^&

  [root@evobot ~]# wc -l test.txt 
  6 test.txt
  [root@evobot ~]# wc -m test.txt 
  122 test.txt
  ```

- `wc -m`选项统计的字数不仅包括字符串，还包括内容里的**换行符**和空格；

- 另外`wc -w [filename]`可以统计文件内容的单词数，统计时以空格区分字符：

  ```bash
  [root@evobot ~]# cat test2.txt 
  this is test file for wc command,
  这个文件用来测试wc命令.
  [root@evobot ~]# wc -w test2.txt 
  8 test2.txt
  [root@evobot ~]# wc -m test2.txt 
  48 test2.txt
  ```

 **这里可以看出，中文连在一起时，不论多少，统计时都只算一个单词。**

### **uniq**命令

- `uniq`命令用来对文件内容去重复，但是只能对连续多行重复的内容去重复：

  ```bash
  [root@evobot ~]# cat test2.txt 
  123
  2
  123
  2
  [root@evobot ~]# uniq test2.txt 
  123
  2
  123
  2
  [root@evobot ~]# vi test2.txt 
  [root@evobot ~]# !cat
  cat test2.txt 
  2
  123
  123
  2
  [root@evobot ~]# !uniq
  uniq test2.txt 
  2
  123
  2
  ```

- 所以一般在使用`uniq`命令时，需要先进行排序，再去重，与`sort`命令一起使用；

  ```bash
  [root@evobot ~]# sort test2.txt 
  123
  123
  2
  2
  [root@evobot ~]# sort test2.txt | uniq
  123
  2
  ```

- `uniq -c`选项可以在去重时统计重复内容的重复次数：

  ```bash
  [root@evobot ~]# sort test2.txt | uniq -c
        2 123
        2 2
  ```


### **tee**命令

- 之前使用的命令，都不会对文件内容进行更改，要么将处理后的内容输出的屏幕，要么使用重定向将内容输出到指定的文件，但输出到文件时，又看不到具体输出了什么内容，所以`tee`命令就是解决这样的问题，它既可以将处理后的内容输出到屏幕，同时也会将内容输出到文件中去；

- `tee`命令的用法就是`tee [filename]`，后面的文件名是要输出内容的保存文件：

  ```bash
  [root@evobot ~]# sort test2.txt | uniq -c | tee result.txt
        2 123
        2 2
  [root@evobot ~]# cat result.txt 
        2 123
        2 2
  ```

  > 可以看到，内容即输出到屏幕上也写入到了文件里。

- `tee -a`选项的作用是将内容追加到文件中：

  ```bash
  [root@evobot ~]# sort test2.txt | uniq -c | tee -a result.txt 
        2 123
        2 2
  [root@evobot ~]# cat result.txt 
        2 123
        2 2
        2 123
        2 2
  ```

### **tr**命令

- `tr`命令是针对字符进行替换操作的命令，如将小写替换为大写，并且支持使用通配符：

  ```bash
  [root@evobot ~]# echo 'evobot blog' | tr 'e' 'E'
  Evobot blog
  [root@evobot ~]# echo 'evobot blog' | tr '[eb]' '[EB]'
  EvoBot Blog
  [root@evobot ~]# echo 'evobot blog' | tr '[a-z]' '[A-Z]'
  EVOBOT BLOG
  ```

- 在使用`tr`命令时，如果要将多个字符替换为同一个字符，替换的字符不能使用`[]`指定，需要使用`tr '[a-z]' 'x'` 的形式：

  ```bash
  [root@evobot ~]# echo 'evobot blog' | tr '[a-z]' '[x]'
  ]]]]]] ]]]]
  [root@evobot ~]# echo 'evobot blog' | tr '[a-z]' 'x'
  xxxxxx xxxx
  [root@evobot ~]# echo 'evobot blog' | tr '[a-z]' '1'
  111111 1111
  ```

### **split**命令

- `split`命令用来对文件内容进行切割，比如将一个大文件切割为小文件，`split`常用选项为`-b`按大小进行分割，默认按字节进行分割，可以指定分割大小的单位，如`M`、`k`，另一个选项则为`-l`按行进行分割：

  ```bash
  [root@evobot ~]# ls -lh secure.log 
  -rw------- 1 root root 19M 4月  24 23:07 secure.log

  [root@evobot ~]# split -b 6M secure.log
  [root@evobot ~]# ls -lh
  总用量 38M
  -rw------- 1 root root  19M 4月  24 23:07 secure.log
  -rw-r--r-- 1 root root 6.0M 4月  24 23:10 xaa
  -rw-r--r-- 1 root root 6.0M 4月  24 23:10 xab
  -rw-r--r-- 1 root root 6.0M 4月  24 23:10 xac
  -rw-r--r-- 1 root root 621K 4月  24 23:10 xad
  # xa开头的文件就是split分割出来的文件
  ```

- `split`分割时可以指定分割出的文件名的前缀，格式为`split -b 1M [filename] [前缀]`:

  ```bash
  [root@evobot ~]# split -b 6M secure.log secure.
  [root@evobot ~]# ls -lh
  总用量 38M
  -rw-r--r-- 1 root root 6.0M 4月  24 23:17 secure.aa
  -rw-r--r-- 1 root root 6.0M 4月  24 23:17 secure.ab
  -rw-r--r-- 1 root root 6.0M 4月  24 23:17 secure.ac
  -rw-r--r-- 1 root root 621K 4月  24 23:17 secure.ad
  -rw------- 1 root root  19M 4月  24 23:07 secure.log
  ```

- 按行切割的`-l`选项的用法如下：

  ```bash
  [root@evobot ~]# wc -l secure.log 
  180663 secure.log
  [root@evobot ~]# split -l 30000 secure.log secure.
  [root@evobot ~]# wc -l *
     30000 secure.aa
     30000 secure.ab
     30000 secure.ac
     30000 secure.ad
     30000 secure.ae
     30000 secure.af
       663 secure.ag
    180663 secure.log
    361326 总用量
  ```

## shell脚本调用的三种方法

- 在shell脚本中调用另一个shell脚本，可以使用`fork`、`exec`、`source`三种方法，三种方法对环境变量以及后续命令执行的影响各不相同，具体的使用异同如下：

  - **`fork`** ：用法为`fork [/path/to/script.sh]`，这种方式在运行时会开启一个`sub-shell(子shell)`，然后在sub-shell中执行调用的脚本，sub-shell执行时，父shell(parent-shell)依然存在，sub-shell执行时从parent-shell继承环境变量，执行完毕后返回到parent-shell执行父级的后续命令，sub-shell中的环境变量不会带回parent-shell中；
  - **`exec`**： 用法为`exec [/path/to/script.sh]`，这种方式不会新开一个sub-shell执行被调用的脚本，而是直接与父脚本在同一个shell中执行，但是被调用的脚本执行完以后，父脚本`exec`行之后的内容就不会再继续执行；
  - **`source`**： 用法为`source [/path/to/script.sh]`，source与exec唯一的不同点在于被调用的脚本执行完之后会继续执行父脚本的后续命令，同时子脚本的环境变量会影响父级环境变量。

  

- 可以通过下面两个脚本体会三种调用之间的不同：

  ```shell
  #!/bin/bash

  A=B
  echo "PID for 1.sh before exec/source/fork:$$"
  export A
  echo "1.sh:\$A is $A"
  case $1 in
      exec)
          echo "using exec..."
          exec ./2.sh ;;
      source)
          echo "using source..."
          . ./2.sh ;;
      *)
          echo "using fork by default..."
          ./2.sh ;;
  esac
  echo "PID for 1.sh after exec/source/fork;$$"
  echo "1.sh: \$A is $A"

  ```
  

  ```shell
  #!/bin/bash

  echo "PID for 2.sh: $$"
  echo "2.sh get \$A=$A from 1.sh"
  A=C
  export A
  echo "2.sh: \$A is $A"
  ```


- 执行结果如下：

  ```bash
  [root@evobot ~]# ./1.sh 
  PID for 1.sh before exec/source/fork:8286
  1.sh:$A is B
  using fork by default...
  PID for 2.sh: 8287
  2.sh get $A=B from 1.sh
  2.sh: $A is C
  PID for 1.sh after exec/source/fork;8286
  1.sh: $A is B	# 子shell的环境变量不影响父shell

  [root@evobot ~]# ./1.sh exec
  PID for 1.sh before exec/source/fork:8363
  1.sh:$A is B
  using exec...
  PID for 2.sh: 8363
  2.sh get $A=B from 1.sh
  2.sh: $A is C	# 父脚本后面的命令没有执行

  [root@evobot ~]# ./1.sh source
  PID for 1.sh before exec/source/fork:8393
  1.sh:$A is B
  using source...
  PID for 2.sh: 8393
  2.sh get $A=B from 1.sh
  2.sh: $A is C
  PID for 1.sh after exec/source/fork;8393
  1.sh: $A is C	# 父脚本命令继续执行，环境变量受子shell影响
  ```
---

