---
title: Linux正则表达式——常用操作练习
author: Evobot
abbrlink: 60aa0a0
date: 2018-05-02 22:44:31
categories: Centos7
tags:
  - Centos
image:
---



# **sed**使用练习

1. 把/etc/passwd 复制到/root/test.txt，用sed打印所有行：

   `sed -n '1,$'p passwd`

2. 打印test.txt的3到10行：

   `sed -n '3,10'p passwd`

3. 打印test.txt 中包含 ‘root’ 的行：

   `sed -n '/root/'p passwd`

4. 删除test.txt 的15行以及以后所有行：

   `sed -n '15,$'p passwd`	

5. 删除test.txt中包含 ‘bash’ 的行：

   `sed -i '/bash/'d passwd`

6. 替换test.txt 中 ‘root’ 为 ‘toor’：

   `sed s/root/toor/g test.txt`

7. 替换test.txt中 ‘/sbin/nologin’ 为 ‘/bin/login’：

   `sed s#/sbin/nologin#/bin/login#g test.txt`

8. 删除test.txt中5到10行中所有的数字：

   `sed -r s#[0-9]##g test.txt`

9. 删除test.txt 中所有特殊字符（除了数字以及大小写字母）：

   `sed -r s#[^0-9a-zA-Z]##g test.txt`

10. 把test.txt中第一个单词和最后一个单词调换位置：

    `sed 's/\(^[a-zA-Z][a-zA-Z]*\)\([^a-zA-Z].*\)\([^a-zA-Z]\)\([a-zA-Z][a-zA-Z]*$\)/\4\2\3\1/' test.txt`

11. 把test.txt中出现的第一个数字和最后一个单词替换位置:

    `sed -r 's/([0-9]+)(.*$)/\2\1/' test.txt `

12. 把test.txt 中第一个数字移动到行末尾：

    `sed -r 's/([0-9]+)(.*$)/\2\1/' test.txt `

13. 在test.txt  20行到末行最前面加 ‘aaa:’：

    `sed -r 's/([0-9]+)(.*$)/\2\1/' test.txt `

## 扩展用法

### 打印文件中特定的某行到某行之间的内容

- 文件test内容如下：

  > ert
  > fff
  > **
  > [abcfd]
  > 123
  > 324
  > 444
  > [rty]
  > **
  > fgfgf

- 需要截取以下几行内容：

  > [abcfd]
  > 123
  > 324
  > 444
  > [rty]

- 使用`sed`命令，用法为`sed -nr '/\[abcfd\]/,/\[rty\]/'p test.txt`。

### sed转换大小写字母

在`sed`中，使用`\u`表示大写，`\l`表示小写。

1. 将每个单词第一个小写字母转换为大写：

   ```bash
   [root@evobot ~]# head -n3 1.log | sed 's/\b[a-z]/\u&/g'
   13:08:24 2018 123 45sdf Nsifud Ninfd 200 12000 Uavdfiya
   13:08:24 2018 123 45sdf Nsifud Ninfd 200 11000 Uavdfiya
   13:08:24 2018 123 45sdf Nsifud Ninfd 230 12000 Uavdfiya
   ```

2. 将所有小写变成大写：

   ```
   [root@evobot ~]# head -n3 1.log | sed 's/[a-z]/\u&/g'
   13:08:24 2018 123 45SDF NSIFUD NINFD 200 12000 UAVDFIYA
   13:08:24 2018 123 45SDF NSIFUD NINFD 200 11000 UAVDFIYA
   13:08:24 2018 123 45SDF NSIFUD NINFD 230 12000 UAVDFIYA
   ```

3. 大写转换为小写：

   ```bash
   [root@evobot ~]# sed 's/[A-Z]/\l&/g' 1.log 
   13:08:24 2018 123 45sdf nsifud ninfd 200 12000 uavdfiya
   13:08:24 2018 123 45sdf nsifud ninfd 200 11000 uavdfiya
   13:08:24 2018 123 45sdf tvubhijn ninfd 230 12000 uavdfiya
   13:08:24 2018 123 45sdf nsifud ninfd 200 11897 uavdfiya
   ```

### sed在文件某一行末尾添加数字

```bash
[root@evobot ~]# sed -r 's/(.*)/\1 123/' 1.log 
13:08:24 2018 123 45sdf nsifud ninfd 200 12000 uavdfiya 123
13:08:24 2018 123 45sdf nsifud ninfd 200 11000 uavdfiya 123
13:08:24 2018 123 45sdf TVUBHIJN ninfd 230 12000 uavdfiya 123
13:08:24 2018 123 45sdf nsifud ninfd 200 11897 uavdfiya 123

[root@evobot ~]# sed -r 's/(^s.*)/\1:123/' 1.txt 
zx cc vv
sai amsomd sad:123
```

### sed删除某关键字的下一行到最后一行

```bash
[root@test200 ~]# cat test
a
b
c
d
e
f

[root@test200 ~]# sed '/c/{p;:a;N;$!ba;d}' test
a
b
c
```

- 定义一个标签a，匹配c，然后N把下一行加到模式空间里，匹配最后一行时，才退出标签循环，然后命令d，把这个模式空间里的内容全部清除。

  if 匹配"c"
  :a
  追加下一行
  if 不匹配"$"
  goto a
  最后退出循环，d命令删除。

### sed打印指定范围包含某个字符串的行

- 用法为`sed  -n '1,100{/abc/p}'  1.txt`：

  ```bash
  [root@evobot ~]# sed -n '1,3{/110/p}' 1.log 
  13:08:24 2018 123 45sdf nsifud ninfd 200 11000 uavdfiya
  ```

---

# **awk**使用练习

1. 用awk 打印整个test.txt （以下操作都是用awk工具实现，针对test.txt）:

   `awk '{print $0}' test.txt`

2. 查找所有包含 ‘bash’ 的行:

   `awk '/bash/' test.txt`

3. 用 ‘:’ 作为分隔符，查找第三段等于0的行:

   `awk -F ':' '$3==0 {print $0}' test.txt`

4. 用 ‘:’ 作为分隔符，查找第一段为 ‘root’ 的行，并把该段的 ‘root’ 换成 ‘toor’ (可以连同sed一起使用):

   `awk -F ':' '$1=="root"' test.txt | sed s/root/toor/g`

5. 用 ‘:’ 作为分隔符，打印最后一段:

   `awk -F ':' '{print $NF}' test.txt`

6. 打印行数大于20的所有行:

   `wk '{if (NR>20) {print $0}}' test.txt`

7. 用 ‘:’ 作为分隔符，打印所有第三段小于第四段的行:

   `awk -F ':' '{if ($3<$4) {print $0}}' test.txt`

8. 用 ‘:’ 作为分隔符，打印第一段以及最后一段，并且中间用 ‘@’ 连接 （例如，第一行应该是这样的形式 ['root@/bin/bash](mailto:'root%40/bin/bash)‘ ）:

   `awk -F ':' '{OFS="@"} {print $1,$NF}' test.txt`

9. 用 ‘:’ 作为分隔符，把整个文档的第四段相加，求和:

   `awk -F ':' '{(sum=sum+$4)}; END {print sum}' test.txt`。

---

# 相关练习

1. 如何把 /etc/passwd 中用户uid 大于500 的行给打印出来？

   `awk -F ':' '{if ($3>500) {print $0}}' /etc/passwd`

2. awk中 NR，NF两个变量表示什么含义？`awk -F ':' '{print $NR}' /etc/passwd`  会打印出什么结果出来？

   `NR`表示行，`NF`表示分段，`wk -F ':' '{print $NR}' /etc/passwd`表示打印和行数相等的分段，超出分段后打印为空。

3. 用grep把1.txt文档中包含'abc'或者‘123’的行过滤出来，并在过滤出来的行前面加上行号.

   `grep -n -e 'abc' -e '123' 1.txt`

   `grep -E -n 'abc|123' 1.txt`

4. grep  -v '^$' 1.txt   这样会过滤出哪些行？

   A: 会过滤所有行，即打印文件内容

5. '.'   '\*' 和 '.*'   分别表示什么含义？'+'和'?'表示什么含义，这五个符号是否可以在grep中使用，是否可以在egrep、sed以及awk中使用？

   A: '.'表示除换行符外的任意字符，'\*'表示前一个字符出现0次或任意多次，'.*'表示任意字符出现任意次；这几个字符均可以在`grep`、`egrep`、`sed`和`awk`中使用。

6. grep 里面用到一个 {} ，它用在什么情况下？

   A：用在使用正则匹配前面的字符出现指定次数时。

7. sed有一个选项，可以直接更改文本文件，是哪个选项？

   A：`-i`选项，例如`sed -i s/123/112233/g 1.txt`

8. `sed -i 's/.*ie//;s/["|&].*//' file`  这条命令表示什么操作呢？

   A：表示在file中将`ie`及其前面所以字符替换为空并且将`"`双引号和`&`号及其后面的所以字符替换为空。

9. 如何删除一个文档中的所有数字或者字母？

   `sed -r 's/[0-9a-zA-Z]//g' 1.txt `

10. 截取日志1.log的第一段(以空格为分隔符), 按数字排序、然后去重，但是需要保留重复的数量如何做？

    `awk '{print $1}' 1.log | sort -n | uniq -c`

11. 使用awk过滤出1.log中第7段(空格分隔)为'200' 并且第8段为'11897'的行。

    `awk '$7==200 && $8==11897' 1.log`

12. 请比较这两个命令的异同： `grep -v '^[0-9]' 1.txt` 和`grep  '^[^0-9]' 1.txt`：

13. awk中的$0表示什么？为什么以下两条命令的$0结果不一致呢？ `awk -F ':' '{print $0}' 1.txt  和 awk -F ':' '$7=1 {print $0}' 1.txt`

14. 使用grep过滤某个关键词时，如何把包含关键词的行连同上面一行打印出来，那下面一行呢？同时上面和下面都打印出来呢？

    A：打印上面一行使用`-B 1`，打印下面一行使用`A 1`，同时打印上下行使用`-C 1`。

---