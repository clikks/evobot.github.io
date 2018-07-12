---
title: shell编程（二）
author: Evobot
date: 2018-07-12 21:30:57
categories: shell基础
tags:
  - shell基础
image:
---



本文主要介绍shell中的逻辑判断相关的知识，包括shell脚本中if判断的用法、逻辑判断符的用法，以及case条件判断的用法和示例。

<!--more-->

---

# shell中的逻辑判断

- 逻辑判断可以根据设定的条件，有选择的执行对应的命令;
- if条件判断有以下几种形式：

1.  `if 条件;then 语句;fi`，例如：

   ```bash
   #!/bin/bash
    
   a=5
   # -gt表示大于
   if [ $a -gt 3 ];then
       echo ok
   fi                      
   ```

   - 在if判断中，条件语句写在`[]`内，并且`[]`内的语句前后必须要有空格，而变量赋值，等于号左右两边不能有空格。

2.  `if 条件;then 语句; else 语句; fi`，例如：

   ```bash
   #!/bin/bash  
                
   a=5          
   if [ $a -gt 5 ];then
       echo ok  
   else         
       echo 'not ok'
   fi           

   ```

3. `if...; then ...; elif ...; then ...; else ...;fi`，例如：

   ```bash
   #!/bin/bash
       
   a=5 
       
   if [ $a -gt 1 ]  
   then
       echo 'a>1'
   # -lt表示小于
   elif [ $a -lt 6 ]
   then
       echo "<6 && >1"
   else
       echo 'not ok'
   fi  

   ```

- 逻辑判断表达式有以下几种：

<style>
table th:first-of-type {
    width: 140px;
}
</style>

| 逻辑判断表达式 |  作用  |
| :-----: | :--: |
|  `-gt`  |  大于  |
|  `-lt`  |  小于  |
|  `-ge`  | 大于等于 |
|  `-le`  | 小于等于 |
|  `-eq`  |  等于  |
|  `-ne`  | 不等于  |

- 逻辑判断语句也可以使用`<>=`的形式，但条件语句需要使用`if (($a<1))`这种形式；

- 同时判断语句中，可以使用`||`和`&&`结合多个条件，`&&`表示并且(and)，`||`表示或(or)，例如：

  ```bash
  if [ $a -lt 5 ] || [ $a -gt 0 ];then ...
  if [ $a -gt 5 ] && [ $b -gt 5 ];then ...
  ```

# 文件目录属性判断

- 判断文件和目录的属性，如文件是否存在，是否为目录，文件是否可写等等，在shell使用下面的判断关键字：

<style>
table th:first-of-type {
    width: 100px;
}
</style>

|    文件目录判断符    |       作用       |
| :-----------: | :------------: |
| `[ -f file ]` | 判断是否存在并且为普通文件； |
| `[ -d file ]` |  判断是否存在并且为目录；  |
| `[ -e file ]` |  判断文件或目录是否存在；  |
| `[ -r file ]` |   判断文件是否可读;    |
| `[ -w file ]` |   判断文件是否可写；    |
| `[ -x file ]` |   判断文件是否可执行。   |



- 示例1，判断文件是否存在，存在则打印文件存在，否则创建文件：

  ```bash
  #!/bin/bash 
              
  f="/tmp/2018-07-12.log"
              
  if [ -f $f ];then
      echo $f exist
  else        
      touch $f
  fi          

  ```

- 示例2，判断是否存在并且是目录，是则打印，否则创建目录：

  ```bash
  #!/bin/bash    
                 
  f="/tmp/testdir"
                 
  if [ -d $f ];then
      echo "$f exist!"
  else           
      mkdir $f   
  fi             

  ```

- 示例3，判断文件是否可读、可写、可执行，判断读写执行权限，是以运行shell脚本的用户身份进行判断的：

  ```bash
  #判断文件是否可读
  #!/bin/bash
    
  f="/tmp/2018-07-12.log"
    
  if [ -r $f ];then
      echo "$f readable"
  fi                    
  ```

  ```bash
  #判断文件是否可写
  #!/bin/bash       
                    
  f="/tmp/2018-07-12.log"
                    
  if [ -w $f ];then 
      echo "$f writeable"                                                                            
  fi                

  ```

  ```bash
  #判断文件是否可执行
  #!/bin/bash      
                   
  f="/tmp/2018-07-12.log"
                   
  if [ -x $f ];then
      echo "$f execable"
  fi       
  ```

  - 实际上在shell中，可以使用命令行的形式写脚本，如：

  ```bash
  #!/bin/bash           
                        
  f="/tmp/2018-07-12.log"
  #文件存在并且是普通文件，则删除                      
  [ -f $f ] && rm -f $f    

  #文件不存在时，创建文件
  [ -f $f ] || touch $f
  ```

  - 判断文件不存在，则使用`!`取反：

  ```bash
  #!/bin/bash

  if [ ! -f $f ];then
      touch $f
  fi
  ```

# if特殊用法

- `if [ -z "$a" ]`表示判断变量a的值是否为空，如果为空，则条件成立；

  ```bash
  #!/bin/bash                   
                                
  cmd=`wc -l /tmp/2018-07-12.lo`
                                
  if [ -z "$cmd" ];then         
      echo "/tmp/2018-07-12.lo not exist"
      exit                      
  else                          
      echo "/tmp/2018-07-12.lo exist"
  fi                            

  ```

- `if [ -n "$a" ]`则与`-z`相反，判断变量是否为空，如果不为空，则条件成立：

  ```bash
  #!/bin/bash                     
                                  
  cmd=`wc -l /tmp/2018-07-12.lo`  
                                  
  if [ -n "$cmd" ];then           
      echo "/tmp/2018-07-12.lo exist"
      exit                        
  else                            
      echo "/tmp/2018-07-12.lo not exist"                                                            
  fi                              

  ```

  ```bash
  $ if [ -n "$b" ]; then echo $b;else echo "b is null";fi   
  b is null

  ```

- if条件判断还可以使用命令结果作为判断条件，例如：

  ```bash
  # grep -w精准匹配单词
  $ if grep -w 'root' /etc/passwd;then echo exist;fi
  root:x:0:0:root:/root:/bin/bash
  exist

  # grep -q不打印匹配结果
  $ if grep -wq 'root' /etc/passwd;then echo exist;fi
  exist

  ```

- `if [ ! -e file ];then`表示文件不存在时会怎样。


# case判断

- case判断的格式如下：

  ```bash
  case 变量名 in 
      value1)
          command
          ;;
      value2)
          command
          ;;
      *)
          command
          ;;
      esac
  ```

- 在case程序中，可以在条件中使用`|`，表示或的意思，如：

  ```bash
  case $var in 
      value1)
          command
          ;;
      value2|value3)
          command
          ;;
      *)
          command
          ;;
       esac
  ```

- 示例，用户输入数字，判断成绩：

  ```bash
  #!/bin/bash                                                       

  #read用来让用户进行输入
  read -p "Please input a number:" n
   
  if [ -z $n ];then
      echo "Please input a number."
      exit 1
  fi
   
  # 替换$n中的数字，如果替换后不为空，则表示输入的内容存在非数字 
  n1=`echo $n|sed 's/[0-9]//g'`
   
  if [ ! -z $n1 ];then
      echo "Please input a number."
      exit 1
  fi
   
  if [ $n -lt 60 ];then
      tag=1
  elif [ $n -ge 60 ] && [ $n -lt 80 ];then
      tag=2
  elif [ $n -ge 80 ] && [ $n -lt 90 ];then
      tag=3
  elif [ $n -ge 90 ] && [ $n -le 100 ];then
      tag=4
  else
      tag=0
  fi
   
  case $tag in
      1)
          echo "不及格"
          ;;
      2)
          echo "及格"
          ;;                                                                                         
      3|4)
          echo "优秀"
          ;;
      *)
          echo "The number range is 0-100."
          ;;
  esac

  ```

  测试：

  ```bash
  $ bash test.sh              
  Please input a number:45
  不及格

  $ bash test.sh
  Please input a number:60
  及格

  $ bash test.sh
  Please input a number:80
  优秀

  $ bash test.sh
  Please input a number:1a3
  Please input a number.

  $ bash test.sh
  Please input a number:104
  The number range is 0-100.

  ```

---

