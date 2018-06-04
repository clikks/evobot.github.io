---
title: Linux正则表达式——awk扩展学习
author: Evobot
categories: Centos7
tags:
  - Linux
  - Centos
abbrlink: 9960c119
date: 2018-05-03 22:24:01
image:
---



# awk中使用外部shell变量

- awk中调用shell的变量，例如`A=44;echo "ABCD" | awk -v GET_A=$A '{print GET_A}'`，`-v`选项用于定义awk的变量参数，这里就是将shell变量`A`赋予`GET_A`，如果要定义多个变量，就需要多个`-v`选项。

- 在脚本中同样也可以让`awk`获取shell变量，如文件filename内容如下：

  ```bash
  1111111:13443253456
  2222222:13211222122
  1111111:13643543544
  3333333:12341243123
  2222222:12123123123
  ```

  脚本如下：

  ```bash
  #!/bin/bash

  sort -n filename | awk -F ':' '{print $1}' | uniq > id.txt

  for id in `cat id.txt`;do
      echo "[$id]"
      awk -v id2=$id -F ':' '$1==id2 {print $2}' filename
  done
  ```

  输出结果：

  ```bash
  [root@evobot ~]# bash awkscript.sh 
  [1111111]
  13443253456
  13643543544
  [2222222]
  13211222122
  12123123123
  [3333333]
  12341243123
  ```

# awk合并文件

- 例如将两个文件中，第一列相同的行合并到同一行，文件内容如下：

  {% codeblock 1.txt %}
  1 aa
  2 bb
  3 ee
  4 ss
  {% endcodeblock %}
  {% codeblock 2.txt %}
  1 ab
  2 cd
  3 ad
  4 bd
  5 de
  {% endcodeblock %}

- 合并后结果为：

  {% codeblock result.txt %}
  1 ab aa
  2 cd bb
  3 ad ee
  4 bd ss
  5 de
  {% endcodeblock %}

- 使用`awk`实现的命令为：

  ```bash
  [root@evobot ~]# awk 'NR==FNR{a[$1]=$2}NR>FNR{print $0,a[$1]}' 1.txt 2.txt 
  1 ab aa
  2 cd bb
  3 ad ee
  4 bd ss
  5 de 
  ```

  > `NR`表示读取的行数，FNR表示读取的当前行数
  > 所以其实NR==FNR 就表示读取2.txt的时候。 同理NR>FNR表示读取1.txt的时候
  > 数组a其实就相当于一个map

- 因为读取了两个文件，当读取第二个文件时，`NR`行数会继续增加，但是当前行数`FNR`会从第二个文件重新计数，如下：

  ```bash
  [root@evobot ~]# awk '{print NR,$0}' 1.txt 2.txt 
  1 1 aa
  2 2 bb
  3 3 ee
  4 4 ss
  5 1 ab
  6 2 cd
  7 3 ad
  8 4 bd
  9 5 de
  [root@evobot ~]# awk '{print FNR,$0}' 1.txt 2.txt 
  1 1 aa
  2 2 bb
  3 3 ee
  4 4 ss
  1 1 ab
  2 2 cd
  3 3 ad
  4 4 bd
  5 5 de
  ```

- 所以判断`NR>FNR`时，表示读取的是第二个文件，就可以打印第二个文件的内容，并且每行后面将第一个文件的`a`数组即第一个文件的`$2`添加到第二个文件后面。

# 将文件多行输出为一行

- 实现将文件内容多行输出为一行使用`cat`或者shell变量都可以实现：

  ```bash
  [root@evobot ~]# a=`cat 1.txt`
  [root@evobot ~]# echo $a
  1 aa 2 bb 3 ee 4 ss

  [root@evobot ~]# cat 1.txt | xargs
  1 aa 2 bb 3 ee 4 ss
  ```

- 使用`awk`实现的命令如下：

  ```bash
  [root@evobot ~]# awk '{printf("%s",$0)}' 1.txt 
  1 aa2 bb3 ee4 ss

  [root@evobot ~]# awk '{printf("%s ",$0)}' 1.txt
  1 aa 2 bb 3 ee 4 ss 
  ```

  - 这里`%s`后面有空格可以格式化输出，没有空格的话，内容将连在一起。

# awk中gsub函数的使用

```bash
awk 'gsub(/www/,"abc")' /etc/passwd  # passwd文件中把所有www替换为abc
awk -F ':' 'gsub(/www/,"abc",$1) {print $0}' /etc/passwd  # 替换$1中的www为abc
```

# awk截取指定多个域为一行

```bash
for j in `seq 0 20`; do
        let x=100*$j
        let y=$x+1
        let z=$x+100
        for i in `seq $y $z` ; do
                awk  -v a=$i '{printf $a " "}' example.txt >>/tmp/test.txt
                echo " " >>/tmp/test.txt
        done
done
```

# grep 或 egrep 或awk 过滤两个或多个关键词

```bash
grep -E '123|abc' filename # 找出文件（filename）中包含123或者包含abc的行
egrep '123|abc' filename   # 用egrep同样可以实现
awk '/123|abc/'  filename # awk 的实现方式
```



> 更多的awk用法，可以参考[awk使用入门](http://www.cnblogs.com/emanlee/p/3327576.html) 

---