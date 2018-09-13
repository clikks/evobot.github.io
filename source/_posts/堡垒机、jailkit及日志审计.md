---
title: 堡垒机、jailkit及日志审计
author: Evobot
date: 2018-09-13 21:34:57
categories: 堡垒机
tags: 
  - jailkit
image:
---

1. 什么是堡垒机
2. 搭建简易堡垒机
3. 安装jailkit实现chroot
4. 日志审计

<!--more-->



# 堡垒机介绍

- 在一个特定网络环境下，为了保障网络和数据不受外界入侵和破坏，而运用各种技术手段实时收集和监控网络环境中每一个组成部分的系统状态、安全事件、网络活动，以便集中报警、及时处理及审计定责；
- 我们又把堡垒机称为跳板机，简易的跳板机功能简单，主要核心功能是远程登录服务器和日志审计；
- 比较优秀的开源软件有jumpserver，认证、授权、审计、自动化和资产管理；
- 商业的堡垒机有齐治、Citrix XenApp。

# 搭建简易堡垒机

## 堡垒机需求

- 具备堡垒机的条件，首先该机器有公网和私网，其中私网和机房其他机器互通；
- 堡垒机安全设置：iptables端口限制、登录限制sshd_config；
- 用户、命令权限限制：jailkit；
- 客户机器日志审计。

## 使用jailkit实现chroot

- 首先到[jailkit官网](https://olivier.sessink.nl/jailkit/index.html#download)下载jailkit2.19版本的安装包；

- 解压jailkit安装包后，执行编译命令`./configure && make && make install`；

- 然后创建`/homr/jail`目录，作为我们的虚拟系统所在位置，接着执行以下命令，将允许执行的命令添加到虚拟系统目录中：

  ```bash
  mkdir /home/jail
  jk_init -v -j /home/jail/ basicshell
  jk_init -v -j /home/jail/ editors
  jk_init -v -j /home/jail/ netutils
  jk_init -v -j /home/jail/ ssh
  ```

- 接着创建用户，并为账户设置密码，需要注意创建的用户名里不能带有jail字样：

  ```bash
  useradd user1
  passwd user1
  ```

- 创建`/home/jail/user/sbin`目录，然后将`/usr/sbin/jk_lsh`文件复制到`/home/jail/user/sbin/`目录下，`jk_lsh`相当于虚拟系统的shell：

  ```bash
  mkdir /home/jail/usr/sbin
  cp /usr/sbin/jk_lsh /home/jail/usr/sbin/
  ```

- 之后为虚拟系统创建用户，执行`jk_jailuser -m -j /home/jail [username]`命令创建用户，这里的用户名需要跟之前在系统中创建的用户名相同：

  ```bash
  jk_jailuser -m -j /home/jail user1
  ```

  ```bash
  [root@vm1 jail]# ls -l /home/jail/
  总用量 0
  lrwxrwxrwx. 1 root root   7 9月  13 22:01 bin -> usr/bin
  drwxr-xr-x. 2 root root  44 9月  13 22:01 dev
  drwxr-xr-x. 2 root root 227 9月  13 22:01 etc
  drwxr-xr-x. 3 root root  22 9月  13 22:54 home
  lrwxrwxrwx. 1 root root   9 9月  13 22:01 lib64 -> usr/lib64
  drwxr-xr-x. 7 root root  70 9月  13 22:52 usr
  
  [root@vm1 jail]# cat /home/jail/etc/passwd 
  root:x:0:0:root:/root:/bin/bash
  user1:x:1003:1003::test:/usr/sbin/jk_lsh
  # 这里的user1用户的shel是jk_lsh,这个shell是无法登录的，所以要将其改成/bin/bash
  
  ```

- 修改user1用户的默认shell：

  ```bash
  [root@vm1 jail]# sed -i 's#/usr/sbin/jk_lsh#/bin/bash#g' /home/jail/etc/passwd 
  [root@vm1 jail]# cat /home/jail/etc/passwd
  root:x:0:0:root:/root:/bin/bash
  user1:x:1003:1003::test:/bin/bash
  
  ```

- 完成后，我们可以使用user1用户ssh登录服务器：

  ```bash
  $ ssh user1@192.168.49.128
  user1@192.168.49.128's password: 
  bash: /usr/bin/id: No such file or directory
  bash: /usr/bin/id: No such file or directory
  [user1@vm1 ~]$ 
  
  ```
  - 这里的登录提示`/usr/bin/id`不存在是因为登录时执行`/etc/profile`文件中存在执行/usr/bin/id的命令，可以忽略这个报错。

- 登录之后我们可以看看user1用户的环境：

  ```bash
  [user1@vm1 ~]$ ls -l /
  total 0
  lrwxrwxrwx. 1 root root   7 Sep 13 14:01 bin -> usr/bin
  drwxr-xr-x. 2 root root  44 Sep 13 14:01 dev
  drwxr-xr-x. 2 root root 227 Sep 13 15:42 etc
  drwxr-xr-x. 4 root root  35 Sep 13 15:42 home
  lrwxrwxrwx. 1 root root   9 Sep 13 14:01 lib64 -> usr/lib64
  drwxr-xr-x. 7 root root  70 Sep 13 14:52 usr
  [user1@vm1 ~]$      
  Display all 115 possibilities? (y or n)
  !          cd         do         fgrep      let        readarray  ssh        unalias
  ./         chmod      done       fi         ln         readonly   suspend    unset
  :          command    echo       for        local      return     sync       until
  [          compgen    egrep      function   logout     rm         tar        vi
  [[         complete   elif       getopts    ls         rmdir      test       wait
  ]]         compopt    else       grep       mapfile    rsync      then       wget
  alias      continue   enable     gunzip     mkdir      scp        time       while
  bash       coproc     esac       gzip       mktemp     sed        times      zcat
  bg         cp         eval       hash       more       select     touch      {
  bind       cpio       exec       help       mv         set        trap       }
  break      date       exit       history    popd       sh         true       
  builtin    dd         export     if         printf     shift      type       
  caller     declare    false      in         pushd      shopt      typeset    
  case       dirs       fc         jobs       pwd        sleep      ulimit     
  cat        disown     fg         kill       read       source     umask      
  [user1@vm1 ~]$ ls /etc/
  bashrc	host.conf  issue	ld.so.conf  nsswitch.conf  profile    resolv.conf
  group	hosts	   ld.so.cache	motd	    passwd	   protocols  services
  
  ```

  - 这就是虚拟的用户环境，在这个环境内，可用命令很少；
  - 在这个环境内，也可以设定alias，将命令别名写在家目录的.bashrc或者.bash_profile中即可。

- 如果要添加多个用户，只需要从上面添加用户的步骤开始配置即可。

- 然后我们需要给用户配置秘钥登录，在用户家目录下创建｀.ssh｀目录，然后在其下创建`authorized_keys`文件，将公钥写入文件，接着**在原系统中**更改`/etc/ssh/sshd_config`中，将`PasswordAuthentication yes`配置改为`no`，关闭密码登录；

- 接着编辑`/etc/hosts.allow`，把允许登录的源IP写入：

  ```bash
  sshd: 192.168.199.0/24 1.1.1.1 10.252.0.22
  ```

- 在`/etc/hosts.deny`中将其他来源IP拒绝掉：

  ```bash
  sshd: ALL
  ```

- 还需要在防火墙中，将不需要的服务端口关闭，可以参考之前的iptables文章；

# 日志审计

- 这里的日志审计，需要在所有被登录的机器上进行配置；

- 首先在`/etc/hosts.allow`中配置只允许跳板机的IP登录，另外建议创建一个与跳板机登录用户同名的用户，不再使用root登录，必要时可以配置给用户sudo权限；

- 然后配置日志审计，按照下面的命令进行配置：

  ```bash
  mkdir /usr/local/records
  chmod 777 !$
  chmod +t !$
  ```

  ```bash
  // vi /etc/profile 添加以下脚本
  if [ ! -d /usr/local/records/${LOGNAME} ];then
  	mkdir -p /usr/local/records/${LOGNAME}
  	chmod 300 /usr/local/records/${LOGNAME}
  fi
  export HISTORY_FILE="/usr/local/records/${LOGNAME}/bash_history"
  export PROMPT_COMMAND='{ date "+%Y-%m-%d %T #### $(who am i | awk "{print \$1\" \"\$2\" \"\$5}") #### $(history 1 | { read x cmd;echo "$cmd";})";} >>$HISTORY_FILE'
  ```

- 之后再用户登录这台机器上时，其所执行的命令，都会记录在`/usr/local/records/[username]/bash_history`文件内。

  ---
