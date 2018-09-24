---
title: ansible安装及使用
author: Evobot
date: 2018-09-24 19:43:21
categories: 自动化运维
tags:
  - ansible
image:
---



主要介绍ansible的安装，以及如何使用ansible远程执行命令、拷贝文件或目录、如何远程执行脚本、管理计划任务。

<!--more-->

---

# ansible介绍与安装

## ansible介绍

- ansible不需要安装客户端，通过sshd进行通信，基于模块工作，模块可以由任何语言进行开发；
- ansible不仅支持命令行使用模块，也支持编写yaml格式的playbook，一句编写和阅读；
- 安装ansible非常简单，centos上可以直接使用yum安装，并且提供有web界面，名为tower，需要付费使用，命令行模式为免费使用；
- redhat已经收购了ansible，所以在redhat系上直接使用yum安装的就是ansible的最新版本；
- 关于按ansible，可以参考[ansible first book](https://ansible-book.gitbooks.io/ansible-first-book/)电子书。

## 安装ansible

- 首先准备两台机器，如vm1和vm2；

- 然后在vm1上使用yum安装ansible：

  ```bash
  yum install -y ansible 
  ```

- 接着配置两台机器的密钥认证，将vm1的公钥写到vm2的authorized_keys文件内；

- 然后编辑`/etc/ansible/hosts`文件，该文件用来定义主机组，如web组、db组等等，在该文件内写入主机组的名字，然后将主机的ip或者hostname(需要在`/etc/hosts`文件内进行配置)写在下面即可，如下：

  ```bash
  [testhost]	#组名自定义
  127.0.0.1
  192.168.199.142
  ```

# ansible的使用

## 远程执行命令

- ansible远程执行命令是针对在`/etc/ansible/hosts`内定义的主机组来执行的，使用`ansible [groupname] -m command -a ‘[cmd]’`,这里`-m`是执行使用的模块，远程执行命令使用`command`模块，然后`-a`指定要执行的命令：

  ```bash
  [root@vm1 ~]# ansible testhost -m command -a 'hostname'
  192.168.199.142 | SUCCESS | rc=0 >>
  vm2
  
  127.0.0.1 | SUCCESS | rc=0 >>
  vm1
  
  ```

- 除了指定主机组，也可以针对一台机器执行命令，将上面的命令中的主机组替换为指定主机的ip或hostname(需在/etc/ansible/hosts中使用的是hostname)即可：

  ```bash
  [root@vm1 ~]# ansible 192.168.199.142 -m command -a 'hostname'
  192.168.199.142 | SUCCESS | rc=0 >>
  vm2
  
  ```

- 如果远程主机开启了selinux，执行远程命令会报错如下信息，只需要安装`libselinux-python`软件包即可：

  ```bash
  “msg":"Aborting,target uses selinux but python bindings (libselinux-python) aren't installed!"
  ```

- 另外absible还有一个shell模块也可以用来执行远程命令，命令为`ansible [group] -m shell -a ‘cmd’`，实际上shell模块是用来远程执行脚本的，但是用来执行单条命令也没有问题：

  ```bash
  [root@vm1 ~]# ansible testhost -m shell -a 'hostname'
  127.0.0.1 | SUCCESS | rc=0 >>
  vm1
  
  192.168.199.142 | SUCCESS | rc=0 >>
  vm2
  
  ```

## 拷贝文件或目录

- ansible拷贝目录和文件使用`copy`模块，命令格式为`ansible [group] -m copy -a "src=[/path/dir] dest=[/path/dir] owner=[username] group=[groupname] mode=[0755]"`，这里的`src`指定源目录，`dest`指定目标目录，`owner`指定属主，`group`指定属组，`mode`指定权限；

- ansible拷贝文件和目录，会将源目录放到指定的目标目录下面去，如果目标目录不存在，将会自动创建新目录，如果拷贝的是文件，`dest`中指定的文件名和源文件名不同，则会重命名，而如果目标路径下存在与文件同名的目录，ansible则会将文件放到同名的目录下面：

  ```bash
  [root@vm1 ~]# ansible vm2 -m copy -a "src=/etc/ansible dest=/tmp/ansible_test owner=root group=root mode=0755"
  vm2 | SUCCESS => {
      "changed": true, 
      "dest": "/tmp/ansible_test/", 
      "src": "/etc/ansible"
  }
  
  ```

  - 查看vm2上拷贝来的目录：

  ```bash
  [root@vm2 tmp]# ll ansible_test/
  总用量 0
  drwxr-xr-x. 3 root root 51 9月  24 23:58 ansible
  
  ```

  - **在指定源目录时，最后的目录路径不能带`/`，否则相当于将源目录下的文件进行拷贝**：

  ```bash
  [root@vm1 ~]# ansible vm2 -m copy -a "src=/root/.ssh/ dest=/tmp/vm1_copy/ owner=root group=root mode=0755"
  vm2 | SUCCESS => {
      "changed": true, 
      "dest": "/tmp/vm1_copy/", 
      "src": "/root/.ssh/"
  }
  
  [root@vm2 tmp]# ll vm1_copy/
  总用量 16
  -rwxr-xr-x. 1 root root  771 9月  24 23:54 authorized_keys
  -rwxr-xr-x. 1 root root 1675 9月  24 23:54 id_rsa
  -rwxr-xr-x. 1 root root  390 9月  24 23:54 id_rsa.pub
  -rwxr-xr-x. 1 root root  868 9月  24 23:54 known_hosts
  
  ```

- 拷贝文件如下：

  ```bash
  [root@vm1 ~]# ansible vm2 -m copy -a "src=/etc/passwd dest=/tmp owner=root group=root mode=0755"
  vm2 | SUCCESS => {
      "changed": true, 
      "checksum": "d2212e95862f22511ab1610cd53326e11482fef9", 
      "dest": "/tmp/passwd", 
      "gid": 0, 
      "group": "root", 
      "md5sum": "dbc0139917c74a337bc390171007ac4a", 
      "mode": "0755", 
      "owner": "root", 
      "secontext": "unconfined_u:object_r:admin_home_t:s0", 
      "size": 1300, 
      "src": "/root/.ansible/tmp/ansible-tmp-1537804974.59-42062188467412/source", 
      "state": "file", 
      "uid": 0
  }
  
  [root@vm2 tmp]# ll passwd 
  -rwxr-xr-x. 1 root root 1300 9月  25 00:02 passwd
  
  ```

- 拷贝文件并重命名：

  ```bash
  [root@vm1 ~]# ansible vm2 -m copy -a "src=/etc/passwd dest=/tmp/vm1_passwd owner=root group=root mode=0755"
  vm2 | SUCCESS => {
      "changed": true, 
      "checksum": "d2212e95862f22511ab1610cd53326e11482fef9", 
      "dest": "/tmp/vm1_passwd", 
      "gid": 0, 
      "group": "root", 
      "md5sum": "dbc0139917c74a337bc390171007ac4a", 
      "mode": "0755", 
      "owner": "root", 
      "secontext": "unconfined_u:object_r:admin_home_t:s0", 
      "size": 1300, 
      "src": "/root/.ansible/tmp/ansible-tmp-1537805059.1-234962579446258/source", 
      "state": "file", 
      "uid": 0
  }
  
  [root@vm2 tmp]# ll vm1_passwd 
  -rwxr-xr-x. 1 root root 1300 9月  25 00:04 vm1_passwd
  
  ```

## 远程执行脚本

- 首先创建一个shell脚本，内容如下：

  ```bash
  #!/bin/bash
  echo `date` > /tmp/ansible_test.txt
  
  ```

- ansible不能够像saltstack一样直接远程执行脚本，其必须先将脚本分发到对应的主机上，然后才能执行，使用ansible拷贝文件的命令将脚本拷贝到对应的主机上：

  ```bash
  [root@vm1 ~]# ansible vm2 -m copy -a "src=/root/test_shell.sh dest=/tmp/test_shell.sh mode=0755"
  vm2 | SUCCESS => {
      "changed": true, 
      "checksum": "1a6e4af02dba1bda6fc8e23031d4447efeba0ade", 
      "dest": "/tmp/test_shell.sh", 
      "gid": 0, 
      "group": "root", 
      "md5sum": "edfaa4371316af8c5ba354e708fe8a97", 
      "mode": "0755", 
      "owner": "root", 
      "secontext": "unconfined_u:object_r:admin_home_t:s0", 
      "size": 48, 
      "src": "/root/.ansible/tmp/ansible-tmp-1537805501.67-208729732802698/source", 
      "state": "file", 
      "uid": 0
  }
  
  ```

- 然后使用shell模块远程执行脚本：

  ```bash
  [root@vm1 ~]# ansible vm2 -m shell -a "/tmp/test_shell.sh"
  vm2 | SUCCESS | rc=0 >>
  
  
  ```

  - 到vm2上查看执行结果：

  ```bash
  [root@vm2 tmp]# cat ansible_test.txt 
  2018年 09月 25日 星期二 00:12:33 CST
  
  ```

- 远程执行命令的时候，我们使用的command模块，这个模块在执行命令的时候不能够在命令中使用管道符，而shell模块支持管道符：

  ```bash
  [root@vm1 ~]# ansible vm2 -m command -a "cat /etc/passwd|wc -l"
  vm2 | FAILED | rc=1 >>
  cat：无效选项 -- l
  Try 'cat --help' for more information.non-zero return code
  
  [root@vm1 ~]# ansible vm2 -m shell -a "cat /etc/passwd|wc -l"
  vm2 | SUCCESS | rc=0 >>
  22
  
  ```

## 管理计划任务

- ansible同样能够管理主机的计划任务，使用`cron`模块，命令格式为`ansible [group] -m cron -a "name='[cron name]' job='[cmd]' (minute|hour|day|month|weekday|)=x"`，这里name指定crontab任务的名字，job则指定执行的命令事什么，后面则是使用ansible的关键字来执行任务执行的时间,不指定的时间则会默认为`*`：

  ```bash
  [root@vm1 ~]# ansible vm2 -m cron -a "name='test cron' job='/bin/touch /tmp/123.txt' minute=0 hour=0 weekday=6"
  vm2 | SUCCESS => {
      "changed": true, 
      "envs": [], 
      "jobs": [
          "test cron"
      ]
  }
  
  ```

- 在vm2上查看计划任务：

  ```bash
  [root@vm2 tmp]# crontab -l
  #Ansible: test cron
  0 0 * * 6 /bin/touch /tmp/123.txt
  
  ```

  - 可以看到指定的任务名以注释的形式被写入crontab任务，这里与saltstack一样，不能自行改动ansible写入的计划任务；

- 删除一个计划任务，使用`ansible [group] -m cron -a "name='cron name' state=absent"`：

  ```bash
  [root@vm1 ~]# ansible vm2 -m cron -a "name='test cron' state=absent"
  vm2 | SUCCESS => {
      "changed": true, 
      "envs": [], 
      "jobs": []
  }
  
  [root@vm2 tmp]# crontab -l
  [root@vm2 tmp]# 
  
  ```

---



