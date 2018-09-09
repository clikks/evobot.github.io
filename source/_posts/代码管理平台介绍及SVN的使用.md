---
title: 代码管理平台介绍及SVN的使用
author: Evobot
categories: 代码管理平台
tags:
  - SVN
abbrlink: f7e78b85
date: 2018-09-07 23:30:24
image:
---

1. 代码管理平台介绍
2.  安装svn
3. 客户端上使用svn

<!--more-->

---

# 代码管理平台介绍

- 代码管理平台主要是为了进行版本控制，记录若干文件内容变化，以便将来查阅特定版本修订情况；
- 版本管理工具发展简史：CVS -> SVN -> GIT；
- SVN全称为subversion，是一个开源版本控制系统，始于2000年；
- GIT是linux创始人lunus发起的，2005年发布，最初的目的是为了更好管理linux内核代码；
- git和svn不同在于git不需要依赖服务端就可以工作，即git是分布式的；
- github是基于git的在线web页面代码托管平台，而gitlab可以人为是开源的github，两者没有直接关系。

# SVN

## 安装及配置SVN服务端

- 由于svn是C/S架构，所以需要先安装服务端，使用`yum install subversion`安装svn服务端；

- 然后创建版本库，版本库就是用来存放项目代码的仓库，为版本库创建一个`svnroot`目录，然后执行以下命令初始化版本库：

  ```bash
  $ svnadmin create /data/svnroot/myproject/
  $ ls /data/svnroot/myproject/
  conf  db  format  hooks  locks  README.txt
  
  ```

- 初始化后，在项目目录myproject中会有一个conf目录，里面有`authz`、`passwd`和`svnserve.conf`三个文件：

  ```bash
  $ ls -l conf/
  总用量 12
  -rw-r--r--. 1 root root 1080 9月   9 01:56 authz
  -rw-r--r--. 1 root root  309 9月   9 01:56 passwd
  -rw-r--r--. 1 root root 3090 9月   9 01:56 svnserve.conf
  ```

- `authz`文件是权限配置文件，`passwd`为密码文件，`svnserve.conf`为仓库配置文件；

- 编辑`authz`文件，写入以下配置：

  ```bash
  # 定义用户组包含哪些用户
  [groups]
  admins = evobot,user1
  
  # '/'表示myproject项目目录，这里这只admin用户组可以读写，*表示所有用户，设置为只读权限
  [/]
  @admins = rw
  * = r
  
  # 这里是另一种配置形式，myproject表示项目名相对路径，':'表示其下的子项目
  [myproject:/]
  user1 = rw
  ```

- `passwd`文件是用来定义用户的密码的文件，如下：

  ```bash
  # 等号左边为用户名，右边则是密码
  [users]
  evobot = mypasswd
  user1 = user1passwd
  
  ```

- 然后编辑`svnserve.conf`文件，主要配置`[general]`配置项：

  ```bash
  [general]
  # anon-access表示匿名用户的权限，none表示无权限
  anon-access = none
  # auth-access表示已鉴权的用户，write则表示可写
  auth-access = write
  # 指定用户密码文件
  password-db = passwd
  # 指定权限控制文件
  authz-db = authz
  # relm表示配置对哪个目录生效，需要配置项目绝对路径
  realm = /data/svnroot/myproject
  
  ```

- 最后使用命令`svnserve -d -r /data/svnroot`启动服务端，其中`-d`表示以daemon启动，`-r`指定版本库路径：

  ```bash
  $ svnserve -d -r /data/svnroot/
  $ ps aux|grep svn
  root       2200  0.0  0.1 162240   660 ?        Ss   02:19   0:00 svnserve -d -r /data/svnroot/
  root       2202  0.0  0.2 112724   972 pts/0    S+   02:19   0:00 grep --color=auto svn
  $ netstat -tlnp |grep svn
  tcp        0      0 0.0.0.0:3690            0.0.0.0:*               LISTEN      2200/svnserve  
  ```

## 使用svn客户端（linux）

- svn的linux客户端也需要安装`subversion`软件包；

- 使用`svn checkout svn://[ip/repository] --username=[username]`命令将服务端的仓库检出到本地：

  ```bash
  $ svn checkout svn://192.168.49.128/myproject --username=evobot
  认证领域: <svn://192.168.49.128:3690> /data/svnroot/myproject
  “evobot”的密码: 
  
  -----------------------------------------------------------------------
  注意!  你的密码，对于认证域:
  
     <svn://192.168.49.128:3690> /data/svnroot/myproject
  
  只能明文保存在磁盘上!  如果可能的话，请考虑配置你的系统，让 Subversion
  可以保存加密后的密码。请参阅文档以获得详细信息。
  
  你可以通过在“/root/.subversion/servers”中设置选项“store-plaintext-passwords”为“yes”或“no”，
  来避免再次出现此警告。
  -----------------------------------------------------------------------
  保存未加密的密码(yes/no)?yes
  取出版本 0。
  
  ```

  - 这里的用户名和密码是明文保存在`/root/.subversion/auth/svn.simple/`目录下的一个子目录当中的，如果将其删除，下次进行检出或更新版本库的操作时，会提示重新输出用户名密码。

- 检出仓库后，会在当前目录下创建与仓库名同名的目录：

  ```bash
  $ ls
  myproject  session.txt
  $ ls -la myproject/
  总用量 4
  drwxr-xr-x. 3 root root   18 9月   9 02:27 .
  dr-xr-x---. 7 root root 4096 9月   9 02:27 ..
  drwxr-xr-x. 4 root root   75 9月   9 02:27 .svn
  
  ```

- 我们在另一台linux机器上再将仓库检出到本地，然后在项目里创建一些文件，然后使用`svn add [path/file]`将文件纳入版本控制，再使用`svn commit -m 'description' `将文件提交到仓库服务器，`-m`选项是为提交添加描述信息：

  ```bash
  $ svn checkout svn://192.168.49.128/myproject --username=user1
  $ cp /etc/fstab .
  $ ls
  fstab
  $ svn add .
  ./    ../   .svn/ 
  $ svn add ./fstab 
  A         fstab
  $ svn commit -m 'add fstab'
  正在增加       fstab
  传输文件数据.
  提交后的版本为 1。
  
  ```

- 再回到之前的linux客户端上，此时仓库中已经由另一个用户user1提交了一个文件，而当前客户端上需要执行`svn update`命令将远程仓库的更新取回到本地来：

  ```bash
  # 可以使用命令简写
  $ svn up
  正在升级 '.':
  A    fstab
  更新到版本 1。
  $ ls
  fstab
  
  ```

- `svn delelte [filename]`可以在本地将本地克隆的仓库中的文件删除，如果需要将删除操作推送到远程仓库，则需要执行`commit`操作；

  ```bash
  # 执行删除操作后，文件标识变成D
  $ svn delete fstab 
  D         fstab
  $ svn commit -m 'delete fstab'
  正在删除       fstab
  
  提交后的版本为 2。
  
  # 在另一个客户端上更新仓库
  $ svn up
  正在升级 '.':
  D    fstab
  更新到版本 2。
  
  ```

- `svn log`则可以看到整个仓库的变更历史：

  ```bash
  $ svn log
  ------------------------------------------------------------------------
  r2 | evobot | 2018-09-09 02:49:57 +0800 (日, 2018-09-09) | 1 行
  
  delete fstab
  ------------------------------------------------------------------------
  r1 | user1 | 2018-09-09 02:37:49 +0800 (日, 2018-09-09) | 1 行
  
  add fstab
  ------------------------------------------------------------------------
  
  ```

## 使用svn客户端（windows）

- 到[https://tortoisesvn.net/index.zh.html](https://tortoisesvn.net/index.zh.html])下载tortoisesvn软件并安装；
- 然后在本地需要存放仓库的目录下点击右键，选择SVN Checkout菜单，然后填写远程仓库的地址，然后点击OK，输入用户名密码即可；
- 在仓库中创建文件，在文件上点右键选择Add，然后选择commit，添加说明后点击OK，图标也会变成绿色的对勾，就完成了文件推送；
- 其他如删除、查看历史，都可以在菜单中选择。

---