---
title: ELK介绍及安装准备
author: Evobot
date: 2019-06-12 21:59:05
categories: ELK
tags: ElasticSearch
image:
---



1. ELK介绍
2. ELK安装准备工作

<!--more-->

---

# ELK介绍

## 需求背景

- 业务发展越来越庞大，服务器越来越多；
- 各种访问日志、应用日志、错误日志量越来越多；
- 开发人员排查问题，需要到服务器上查日志，不方便；
- 运营人员需要一些数据，需要运维到服务器上分析日志。

## ELK相关介绍

- [ELK中文官网](https://www.elastic.co/cn/)对ELK相关产品有相应的介绍，ELK在5.0版本前叫ELK Stack，5.0版本之后为Elastic Stack，包含了ELK Stack和Beats两个套件；
- 关于ELK，可以参考gitbook上的[ELKstack中文指南](https://elkguide.elasticsearch.cn/)；
- ELK stack套件包含：ElasticSearch、Logstash、Kibana三个软件；
- 其中ElasticSearch是一个搜索引擎，用来搜索、分析、存储日志，为分布式架构，可以横向扩容，可以自动发现、索引自动分片、功能很强大；相关文档，可以参考[Elasticsearch:权威指南](https://www.elastic.co/guide/cn/elasticsearch/guide/current/index.html)；
- Logstash用来采集日志，把日志解析为json格式交给ElasticSearch；
- Kibana则是一个数据可视化组件，把处理后的结果通过web界面进行展示；
- Beats则是一个轻量级日志采集器，实际上Beats家族有5个成员；
- 早期的ELK架构中使用Logstash收集、解析日志，但Logstash对内存、cpu、io等资源消耗较高，相比Logstash，Beats所占系统的CPU和内存几乎可以忽略不计，Logstash可以定义输出日志的格式，例如nginx访问日志的格式可以划分为多个字段，如访问时间、主机名、访问的URL等，但Beats是无法做到该功能的；
- x-pack对Elastic Stack提供了安全、警报、监控、报表、图表于一身的扩展包，x-pack是收费的；
- ELK的具体架构图如下：

![ELK架构图](https://s2.ax1x.com/2019/06/12/VW1wOs.png)

# ELK安装准备工作

## 机器划分

- 首先准备三台机器，IP分别为128、130、131；
- 三台机器上都需要安装elasticsearch（后续称为es），1台主节点128，2台数据节点129、130；
- es主128上安装kibana；
- 1台es数据节点129上安装logstash；
- 三台机器全部安装jdk8（openjdk即可）；

## 安装准备

- 在三台机器的hosts文件中定义IP地址对应的hostname：

  ```bash
  192.168.139.128 centos_1
  192.168.139.129 centos_2
  192.168.139.130 centos_3
  
  ```

- 然后在三台机器上安装jdk1.8：

  ```bash
  yum install -y java-1.8.0-openjdk
  ```

---

