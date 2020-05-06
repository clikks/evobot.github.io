---
title: Vim基础(二)
author: Evobot
categories: Centos7
tags:
  - Centos
abbrlink: 49f9b403
date: 2018-04-14 01:42:17
image:
---



![vim模式关系](https://s1.ax1x.com/2020/04/28/JIG5ZV.png)

本文主要介绍Vim的编辑模式相关操作以及命令模式的相关操作。

<!--more-->

---



# 编辑模式

- 想要对文档的内容进行增加和编辑，在vim中需要进入编辑模式，进入编辑模式的常用操作如下表：


<style>
table th:first-of-type {
    width: 180px;
}
table th {
    text-align: center;
}
</style>


| 按键操作 |        作用        |
| :--: | :--------------: |
|  i   |    在当前光标位置插入     |
|  I   |    在光标所在行行首插入    |
|  o   |   在光标所在行下方插入一行   |
|  O   |    在光标所在行上方插入    |
|  a   | 在当前光标位置后一个字符位置插入 |
|  A   |  在当前光标所在行行尾插入编辑  |

- 进入编辑模式后的编辑操作与普通编辑器操作相同。

# 命令模式

- vim的命令模式可以实现对文档保存，退出，搜索或是其他更加复杂的操作，命令模式的相关操作如下表：

<style>
table th:first-of-type {
    width: 180px;
}
table th {
    text-align: center;
}
</style>

| 按键操作                  | 作用                                       |
| -------------------- | --------------------------------------- |
| /word                 | 向光标之后查找一个字符串word，按n向后继续搜索                |
| ?word                 | 向光标之前查找一个字符串word，按n向前继续搜索                |
| :n1,n2s/word1/word2/g | 将n1到n2的行中的word1全部替换为word2，结尾不加g，则只替换每一行第一个word1 |
| :1,$s/word1/word2/g   | 将文档中所有的word1替换为word2，当word1中存在`/`时，可以使用`:1,$s#word1#word2#g` |
| :w                    | 保存文本                                     |
| :q                    | 退出vim                                    |
| :w!                   | 强制保存，在root用户下，即使文本只读也可以保存                |
| :q!                   | 不保存改动并强制退出                               |
| :wq                   | 保存并退出，没有改动时也会修改文档mtime                   |
| :set nu               | 显示行号                                     |
| :set nonu             | 取消显示行号                                   |
| :x                    | 未改动文档内容时，保存退出且不改动文档mtime                 |

---

