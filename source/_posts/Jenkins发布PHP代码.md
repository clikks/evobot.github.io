---
title: Jenkins发布PHP代码
author: Evobot
categories: 持续集成
tags:
  - Jenkins
abbrlink: aac66d25
date: 2019-04-23 21:54:01
image:
---



本文主要介绍如何配置Jenkins的插件，然后使用Jenkins创建任务，通过插件，完成从GIT仓库拉取代码并发布到指定的远程机器上。

<!--more-->

---

# Jenkins发布PHP代码

## 插件安装

- 首先登录到Jenkins的web界面，在“系统管理” -> “插件管理”页面，可以看到四个标签，分别是可更新、可选插件、已安装插件以及高级标签，其中高级标签是在Jenkins无法联网是，配置代理让Jenkins联网的功能；
- 我们需要点击“已安装”标签，然后检查是否安装有`Git plugin`和`Publish Over SSH`两个插件，如果没有安装，则点击“可选插件”标签，找到插件进行安装。

![publish_over_ssh](https://s2.ax1x.com/2019/04/23/EEgQ9H.png)

- 在插件安装界面最下方，有一个选项为“安装完成后重启Jenkins(空闲时)”，对于我们需要立即使用的情况来说，直接在后台使用`systemctl restart jenkins`命令重启Jenkins服务即可：

![restart_jenkins](https://s2.ax1x.com/2019/04/23/EEgh8J.png)

## 插件配置

- 重启Jenkins服务后，重新登录web界面，在“系统管理” -> “系统设置”菜单中，找到插件**Publish Over SSH**的配置项，这里需要配置SSH的密钥，如果机器上之前没有生成过密钥对，就需要使用`ssh-keygen -f /root/.ssh/jenkins` 来生成一对密钥，如果已经生成过密钥，那么也可以直接将私钥填入**Key**菜单的输入框内：

![ssh_key](https://s2.ax1x.com/2019/04/23/EE2MZV.png)

- 然后在下面的"SSH Servers"选项中点击“新增”，这里增加的是被发布PHP代码的SSH Server，需要填写PHP主机的Hostname、Username和Remote Directory远程登录目录，其中Remote Directory建议配置为根目录，配置如下：

![jenkins_ssh_servers](https://s2.ax1x.com/2019/04/23/EERKld.png)

- 上面已经在Jenkins中配置了私钥，然后还需要将Jenkins主机的公钥配置到发布PHP代码的主机的`authorized_keys`文件中去，配置完成后，在Jenkins的SSH Servers配置中点击`Test Congfiguration`按钮测试配置是否正确：

![Jenkins_test_configuration](https://s2.ax1x.com/2019/04/23/EERtfg.png)

- 提示Success后，就可以点击“应用”按钮完成配置，如果需要配置多台Server，则继续新增即可。

## 任务配置

- 在配置完Publish Over SSH插件之后，在Jenkins的web界面上，点击“新建任务”，在弹出的页面中，最上面先输入任务的名字，然后下面选择“构建一个自由风格的软件项目”，然后点击“确定”：

![new_task](https://s2.ax1x.com/2019/04/23/EEW3v9.png)

- 然后弹出的界面中，就是让我们来定义整个任务，定义如何发布PHP的代码，首先填写想要的描述，然后在“源码管理”中点击Git，如果企业使用的是SVN，则选择Subversion，这里我们点击Git，在右侧弹出的输入框中填入项目的git地址，而“Credentials”则选择无即可，这是针对私有仓库的设置项：

![task_setting](https://s2.ax1x.com/2019/04/23/EEfVRe.png)

- 在"Branches to build"选项中，填写需要发布的代码分支，一般保持默认的master即可，接着在下面的“构建”菜单下，选择“增加构建步骤“下拉框，在下拉框内选择`Send files or execute commands over SSH`，弹出的“SSH Server”对话框中，首先选择要发布的远程主机的名字，然后在“Transfers”菜单中的“Source files”中填写`**/**`，表示发布全部代码，而"Remote directory"中则是填写远程主机代码存放的路径，而"Exec command"是指文件传输完成后要执行的命令，这里填写修改目录属主和属组的命令`chown -R nobody:nobody /tmp/jenkins_php`"Remove prefix"则是指定截掉的前缀目录，这里不需要填写：

![task_setting2](https://s2.ax1x.com/2019/04/23/EEhR1g.png)

- 同样的，"构建"菜单也可以增加多个主机和传输任务，完成后，点击"保存"即可。

## 构建任务

- 完成任务配置之后，返回到工作台面板，可以看到右侧列出了所有的任务：

![task_list](https://s2.ax1x.com/2019/04/23/EEh5Bn.png)

- 然后我们点击任务的名称，进入到任务详情里，接着点击左边菜单的"立即构建"，Jenkins就会根据任务配置，开始构建我们的项目，将PHP代码发布到远程主机上：

![task_detail](https://s2.ax1x.com/2019/04/23/EE4F3D.png)



- 在构建任务的过程中，左下角的"构建执行状态"中会列出正在构建的项目，点击正在构建的任务序号的小三角，会弹出菜单，选择"控制台输出"就可以查看整个构建的详细过程：

![task_building](https://s2.ax1x.com/2019/04/23/EE4JDs.png)

![console_output](https://s2.ax1x.com/2019/04/23/EE4a5V.png)

- 从控制台输出上看，构建已经成功，可以到远程主机上查看目录：

  ```bash
  [root@vm2 ~]# cd /tmp/jenkins_php/
  [root@vm2 jenkins_php]# ls -l
  总用量 8
  -rw-r--r--. 1 nobody nobody  352 4月  23 23:34 manage.py
  drwxr-xr-x. 3 nobody nobody   91 4月  23 23:27 migrations
  -rw-r--r--. 1 nobody nobody 1011 4月  23 23:34 requirements.txt
  drwxr-xr-x. 3 nobody nobody  102 4月  23 23:27 scripts
  drwxr-xr-x. 5 nobody nobody  157 4月  23 23:27 simpledu
  
  ```

- Jenkins发布代码，如果Git仓库上的代码发生了改动，那么我们只需要重新点击构建，Jenkins就会将Git仓库内的新的代码发布到远程主机上。

# Jenkins构建wordpress

- 利用jenkins自动发布wordpress代码，首先使用git将代码推送到git仓库，然后在jenkins中创建任务如下：

![jenkins_wordpress](https://s2.ax1x.com/2019/05/12/E49EaF.png)

![jenkins_wordpress2](https://s2.ax1x.com/2019/05/12/E49ePJ.png)

- 然后保存任务，点击任务名右侧下拉箭头选择立即构建，构建成功后，控制台输出如下：

![jenkins_wordpress3](https://s2.ax1x.com/2019/05/12/E49Qr6.png)

- 在实际应用中，可能会因为一些问题需要进行回滚操作，所以在构建完成后，可以对任务进行修改，在jenkins自动构建之前，进行代码的备份操作，在构建菜单中，修改`Exec command`的内容如下：

  ```bash
  d=`date +%Y%m%d%H%M` && tar zcfP /data/wordpress_old_$d.tar.gz /data/wordpress/  && chown -R apache:apache /data/wordpress
  ```

- 保存之后，每一次构建，都会以日期为后缀对代码目录进行打包备份。

- 对于需要回滚的情况，需要新建一个任务，只增加构建步骤，选择`Send files or execute commands over SSH`，选择SSH Server，然后在Exec command对话框中，填入下面的命令：

  ```bash
  d=`date +%Y%m%d%H%M` && mv -f /data/wordpress /data/wordpress_failed_$d && tar zxfP `ls -tl /data/*.tar.gz|awk 'NR==2{print $NF}'`
  
  ```

- 保存任务后，执行构建，就会将wordpress的代码回滚到上一个版本。

---



