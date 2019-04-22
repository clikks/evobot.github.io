---
title: Jenkins安装与介绍
author: Evobot
date: 2019-04-22 21:53:01
categories:  持续集成
tags:
  - Jenkins
image:
---

1. jenkins介绍
2.  jenkins安装
3. 了解jenkins

<!--more-->

---

# Jenkins介绍

- 互联网产品通过产品设计成型 -> 开发人员开发代码 -> 测试人员测试功能 -> 运维人员发布上线的流程进行上线发布；
- 在开发人员开发完成代码后，各个模块的代码需要进行集成，而软件迭代的过程中，需要不停的集成，所以也称之为持续集成（Continuous integration简称CI）；
- 集成完成后需要交付给测试人员进行测试，同样也需要继续进行持续交付（Continuous delivery）；
- 成型的代码需要运维人员部署到线上提供给用户使用，称之为持续部署（Continuous deployment）；
- Jenkins就是一个开源的，可扩展的持续集成、交付、部署（软件/代码的编译、打包、部署）基于web界面的平台，官网为[jenkins.io](jenkins.io)；
- Jenkins是一个工具集，提供了各种各样的插件，比如获取git上的最新代码，帮助编辑源代码，调用自定义的shell脚本远程执行命令等等。

# Jenkins安装

- Jenkins的安装，所需的最低配置为不少于256M内存，不低于1G磁盘，JDK版本要大于等于8；

- 安装Jenkins前，首先要安装jdk，可以参考前面的博文[安装 jdk 和 Tomcat](https://www.evobot.cn/post/af3c490c.html)，或者使用yum安装openjdk：

  ```bash
  yum install -y java-1.8.0-openjdk
  ```

- 然后安装Jenkins的yum源：

  ```bash
  wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat/jenkins.repo
  ```

- 安装了yum源后，还需要导入yum的密钥：

  ```bash
  rpm --import https://pkg.jenkins.io/redhat/jenkins.io.key
  ```

- 接着使用yum安装Jenkins即可，如果安装速度太慢，也可以到https://pkg.jenkins.io/redhat/下载rpm包，然后使用`yum localinstall`命令安装：

  ```bash
  yum install -y jenkins
  ```

- 安装完成后，使用`systemctl start jenkins`启动jenkins；

  ```bash
  [root@localhost ~]# systemctl start jenkins
  [root@localhost ~]# ps aux |grep jenkins
  jenkins    3170 69.2 32.7 2238276 157860 ?      Ssl  23:17   0:11 /etc/alternatives/java -Dcom.sun.akuma.Daemon=daemonized -Djava.awt.headless=true -DJENKINS_HOME=/var/lib/jenkins -jar /usr/lib/jenkins/jenkins.war --logfile=/var/log/jenkins/jenkins.log --webroot=/var/cache/jenkins/war --daemon --httpPort=8080 --debug=5 --handlerCountMax=100 --handlerCountMaxIdle=20
  
  ```

- 第一次启动后，需要查看`/var/log/jenkins/jenkins.log`文件，该文件记录了jenkins安装启动的所有日志，并且会提供一个初始化安装的密码，同时也提示可以在/var/lib/jenkins/secrets/initialAdminPassword文件内找到密码：

  ```
  Jenkins initial setup is required. An admin user has been created and a password generated.
  Please use the following password to proceed to installation:
  
  d84d9f50c4014e28a84df1ef26798b96
  
  This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
  
  ```

- 启动jenkins后，访问`ip:8080`访问jenkins：

![unlock_jenkins](https://s2.ax1x.com/2019/04/22/EAlXGj.png)

- 在页面上输入上面查看到的密码进入jinkens，会有一个等待初始化的过程，然后会提示安装插件，选择安装建议安装的插件即可：

![install_plugin](https://s2.ax1x.com/2019/04/22/EA1eQ1.png)

![install_plugin2](https://s2.ax1x.com/2019/04/22/EA1msx.png)

# 了解Jenkins

- 在完成Jenkins的插件安装之后，会提示要创建管理员用户，按提示创建并选择保存并完成即可；
- 进入jenkins工作台的界面如下：

![jenkins_platform](https://s2.ax1x.com/2019/04/22/EA3oE8.png)

- jenkins的工作台界面非常简单，如果要实现持续集成、交付、部署，还需要借助很多插件来构建任务；

- 在`/etc/sysconfig/`目录下，有名为`jenkins`的配置文件，该文件内有jenkins的基础配置，例如定义jenkins程序目录的`JENKINS_HOME`配置、定义启动jenkins用户身份的`JENKINS_USER`配置等等：

  ```ini
  # Directory where Jenkins store its configuration and working
  # files (checkouts, build reports, artifacts, ...).
  #
  JENKINS_HOME="/var/lib/jenkins"
  
  ## Type:        string
  ## Default:     ""
  ## ServiceRestart: jenkins
  #
  # Java executable to run Jenkins
  # When left empty, we'll try to find the suitable Java.
  #
  JENKINS_JAVA_CMD=""
  
  ## Type:        string
  ## Default:     "jenkins"
  ## ServiceRestart: jenkins
  #
  # Unix user account that runs the Jenkins daemon
  # Be careful when you change this, as you need to update
  # permissions of $JENKINS_HOME and /var/log/jenkins.
  #
  JENKINS_USER="jenkins"
  
  ```

- 进入Jenkins的程序目录`/var/lib/jenkins`：

  ```bash
  [root@localhost ~]# cd /var/lib/jenkins/
  [root@localhost jenkins]# ls
  config.xml                                      nodeMonitors.xml
  hudson.model.UpdateCenter.xml                   nodes
  hudson.plugins.git.GitTool.xml                  plugins
  identity.key.enc                                secret.key
  jenkins.install.InstallUtil.lastExecVersion     secret.key.not-so-secret
  jenkins.install.UpgradeWizard.state             secrets
  jenkins.model.JenkinsLocationConfiguration.xml  updates
  jenkins.telemetry.Correlator.xml                userContent
  jobs                                            users
  logs                                            workflow-libs
  
  ```

- 在上面的目录下，jobs是存放jenkins上创建的任务的目录，nodes则是多节点使用的目录，plugins则是存放插件的目录，secrets是存放密码的目录，users是保存用户的目录，在使用jenkins的过程中，我们主要关注的是plugins目录和jobs的目录；

- 如果需要备份jenkins，可以直接拷贝`/var/lib/jenkins`目录到新的机器上，jenkins运行不需要借助数据库，所有的数据都存放在`jenkins`目录下的xml文件内。

---

