---
title: exportfs命令、NFS客户端问题及FTP服务搭建
author: Evobot
categories:
  - NFS
  - FTP
tags:
  - NFS
  - FTP
abbrlink: 38114ae5
date: 2018-06-22 22:11:38
image:
---



本文介绍了exportfs命令管理nfs配置文件，以及NFS4版本中存在的用户权限问题，最后一部分则是FTP服务的介绍、以及如何使用vsftpd软件包搭建ftp服务。

<!--more-->

---

# exportfs命令

- `exportfs`命令实际上和nfs-utils软件包一起安装的，exportfs命令的作用是不中断nfs服务重新生效配置文件；
- `exportfs`主要使用的有以下几个选项：

  - `-a`全部挂载或全部卸载；
  - `-r`重新挂载；
  - `-u`卸载某一个目录；
  - `-v`显示共享目录。
- 常用选项`exportfs -arv`，即可不中断服务重新生效配置文件；


示例如下：

- 首先在服务端的`/etc/exports`文件内，增加一个要共享的目录：

  ```bash
  [root@localhost ~]# cat /etc/exports
  /home/nfsdir 192.168.67.0/24(rw,sync,all_squash,anonuid=1000,anongid=1000)
  /usr/local/src 192.168.67.0/24(rw,sync,no_root_squash)
  ```

  > 使用了`no_root_squash`，即不限制root，这样使用root用户创建的文件，在服务端和客户端显示权限都是root。

- 然后在服务端执行命令`exportfs -arv`：

  ```bash
  [root@localhost ~]# exportfs -arv
  exporting 192.168.67.0/24:/usr/local/src
  exporting 192.168.67.0/24:/home/nfsdir

  ```

- 在服务端使用`showmount`命令查看是否新增了新的共享目录：

  ```bash
  [root@localhost ~]# showmount -e 192.168.67.128
  Export list for 192.168.67.128:
  /usr/local/src 192.168.67.0/24
  /home/nfsdir   192.168.67.0/24
  ```

- 最后挂载新的共享目录，并尝试创建文件，查看文件是否能够实时更新到服务端：

  ```bash
  #客户端创建文件
  [root@localhost ~]# touch /media/test2
  [root@localhost ~]# ls /media/
  mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz  php-7.1.6  php-7.1.6.tar.bz2  test2

  #服务端查看文件
  [root@localhost ~]# ls /usr/local/src/
  mysql-5.7.17-linux-glibc2.5-x86_64.tar.gz  php-7.1.6  php-7.1.6.tar.bz2  test2

  ```

# NFS客户端问题

- NFS4版本存在客户端挂载共享目录后，不论时root还是普通用户，创建新文件时，属组、属主都会变为nobody的问题；

- 解决这个问题可以在挂载的时候使用nfs3版本，命令如下：

  ```bash
   mount -t nfs -o nfsvers=3 192.168.67.128:/home/nfsdir /tmp/nfs/
  ```

  > 使用mount的`-o`参数指定nfs的版本为3

  或者使用remount参数重新挂载：

  ```bash
  mount -t nfs -oremount,nfsvers=3 192.168.67.128:/home/nfsdir /tmp/nfs/
  ```

- 另外一种解决方法，是编辑`/etc/idmapd.conf`文件，将`#Domain = local.domian.edu`改为`Domain = xxx.com`，其中的xxx.com可以自定义，然后重启`rpcbidmapd`服务即可。


---

# FTP服务搭建

## FTP服务介绍

- FTP是File Transfer Protocol文件传输协议的英文简称，主要用于在Internet上控制文件的双向传输；
- FTP的主要作用就是让客户端连接一个FTP服务端的远程计算机，并查看远程计算机中的文件，并且能够吧文件下载到本地，或上传到远程计算机；
- FTP因为安全性不高，所以在大型企业使用较少。

## 使用vsftpd搭建ftp

- 使用`yum install -y vsftpd`安装vsftpd软件包，vsftpd能够使用系统的用户进行登录ftp服务，默认会进入登录的用户的家目录，而这种方式不够安全；

- 所以我们一般会为vsftpd创建虚拟用户，并且将虚拟用户映射为系统的普通用户，这样即使将虚拟用户密码交给其他人，也不能够使用这个账户和密码登录ssh服务；

1. 首先添加一个普通用户，这个普通用户是用来给虚拟用户做映射，使得虚拟用户在传输时，能够有用户身份去操作文件；

  ```bash
  useradd -s /sbin/nologin virftp
  ```

2. 然后创建`/etc/vsftpd/vsftpd_login`虚拟用户密码文件，并在文件内写入用户名和密码，其中奇数行为用户名，偶数行为密码，多个用户写多行：

  ```bash
  evobot
  password
  user2
  password2
  ```

3. 然后更改`vsftpd_login`文件权限为600：

  ```bash
  chmod 600 /etc/vsftpd/vsftpd_login
  ```

4. 然后还需要将这个文本的密码文件转换为二进制的密码文件，使用`db_load`命令：

  ```bash
  db_load -T -t hash -f /etc/vsftpd/vsftpd_login /etc/vsftpd/vsftpd_login.db
  ```

5. 接着创建虚拟用户配置文件存储目录：

  ```bash
  mkdir /etc/vsftpd/vsftpd_user_conf
  ```

6. 在`vsftp_user_conf`目录下创建与**虚拟用户同名**的配置文件：

   ```bash
   touch /etc/vsftpd/vsftpd_user_conf/evobot
   ```

   然后在上面创建的文件中写入以下内容：

   ```bash
   # 定义虚拟用户的家目录，即登录ftp时的目录
   local_root=/home/virftp/evobot
   # 是否允许匿名用户
   anonymous_enable=NO
   # 是否允许可写
   write_enable=YES
   # 创建文件及目录时的权限
   local_umask=022
   # 是否允许匿名用户可上传
   anon_upload_enable=NO
   # 是否允许匿名用户可写权限
   anon_mkdir_write_enable=NO
   # 空闲会话超时时间
   idle_session_timeout=600
   # 数据传输超时时间
   data_connection_timeout=120
   # 最大客户端数
   max_clients=10
   ```

7. 创建local_root虚拟用户家目录，并将目录属主和属组改为virftp映射用户：

   ```bash
   mkdir /home/virftp/evobot

   chown -R virftp.virftp /home/virftp/evobot/
   ```

8. 修改`/etc/pam.d/vsftpd`文件，这个文件是vsftpd用来进行认证的文件，文件中规定了认证的文件和认证的形式，在文件最前面加上下面的内容：

   ```bash
   auth sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd_login
   account sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd_login

   ```

   > 这里需要注意/lib64/security/pam_userdb.so文件在32位系统中，是在/lib/security/pam_userdb.so路径。

   文件最终内容如下：

   ```bash
   #%PAM-1.0
   auth sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd_login
   account sufficient /lib64/security/pam_userdb.so db=/etc/vsftpd/vsftpd_login
   session    optional     pam_keyinit.so    force revoke
   auth       required     pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
   auth       required     pam_shells.so
   auth       include      password-auth
   account    include      password-auth
   session    required     pam_loginuid.so
   session    include      password-auth

   ```

9. 最后编辑vsftpd的主配置文件`/etc/vsftpd/vsftpd.conf`：

   - 将***anonymous_enable=YES***改为NO；
   - 将***#anon_upload_enable=YES***改为**anon_upload_enable=NO**;
   - 将***#anon_mkdir_write_enable=YES***改为**anon_mkdir_write_enable=NO**
   - 然后增加以下内容：

   ```bash
   chroot_local_user=YES
   # 定义映射的普通用户
   guest_enable=YES
   guest_username=virftp
   virtual_use_local_privs=YES
   # 定义虚拟用户配置文件目录
   user_config_dir=/etc/vsftpd/vsftpd_user_conf
   allow_writeable_chroot=YES
   ```

10. 然后启动vsftpd服务：

  ```bash
  [root@localhost ~]# systemctl start vsftpd
  [root@localhost ~]# ps aux| grep vsftpd
  root       2904  0.0  0.1  53260   576 ?        Ss   00:33   0:00 /usr/sbin/vsftpd /etc/vsftpd/vsftpd.conf

  [root@localhost ~]# netstat -tlnp | grep vsftpd
  tcp6       0      0 :::21                   :::*                    LISTEN      2904/vsftpd  
  ```

## 测试vsftpd服务

- 在linux上，安装lftp客户端软件，然后使用命令`lftp evobot@127.0.0.1`登录本地的FTP服务：

  ```bash
  [root@localhost evobot]# lftp evobot@127.0.0.1
  口令: 
  lftp evobot@127.0.0.1:~> ls         
  -rw-r--r--    1 0        0               0 Jun 22 16:36 test
  lftp evobot@127.0.0.1:/> 

  ```

- 在lftp中，使用`?`会显示帮助信息，如`get`命令进行下载文件到当前目录下。

