---
title: shell习题-每日生产一个文件
author: Evobot
date: 2018-07-12 15:08:24
categories: shell习题
tags:
  - shell习题
image:
---

Q:需求每天将磁盘使用情况写入到以日期命名的log文件内，日期格式xxxx-xx-xx，后缀名.log。

A：脚本如下

  ```bash
  #!/bin/bash
  
  d=`date +%F`
  file_name=$d.log
  df -h > /tmp/$file_name
  ```