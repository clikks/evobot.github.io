---
title: Jenkins部署java项目
author: Evobot
date: 2019-05-16 22:19:51
categories: 持续集成
tags: Jenkins
image:
---



1. 部署java项目-创建私有仓库
2. 部署java项目-下载zrlog源码
3. 安装配置tomcat
4. 部署java项目-安装maven
5. 安装插件
6. 构建job
7. 手动安装jdk
8. 发布war包

<!--more-->

---

# Jenkins部署java项目

## 创建私有仓库

- Jenkins的使用主要以java的持续集成为主，因为Java需要编译和打包，而编译打包Java项目，使用maven完成，所以需要安装maven；
- 编译完成后将Java项目打包，然后部署到tomcat中；
- 首先在github或gitlab中创建项目，项目名称自定义，然后项目类型选择私有：

![jenkins_java_gitlab](https://s2.ax1x.com/2019/05/16/Eb5c6K.png)

- 在gitlab新窗口的用户设置内的 SSH Keys 中添加自己机器的ssh公钥：

![jenkins_java_gitlab2](https://s2.ax1x.com/2019/05/16/Eb5O0g.png)

- 配置完密钥后，按照gitlab项目内的提示，将仓库克隆到本地：

![jenkins_java_gitlab3](https://s2.ax1x.com/2019/05/16/EbIKc6.png)

- 如果你的gitlab部署的ssh端口不是默认的22端口，那么还需要在`.ssh`目录下创建`config`文件，写入以下内容来指定gitlab的ssh端口号：

  ```ini
  Host git.evobot.cn
  User git
  Port 8022
  ```

- 将仓库克隆到本地：

  ```bash
  [root@localhost ~]# git clone git@git.evobot.cn:evobot/java_project.git
  正克隆到 'java_project'...
  The authenticity of host '[git.evobot.cn]:8022 ([192.54.103.130]:8022)' can't be established.
  ECDSA key fingerprint is SHA256:GaeVoziRC1R0hxySNMyWyfDNntWpD1dxDmvbLr5NVF4.
  ECDSA key fingerprint is MD5:ec:e6:db:a1:c9:6b:db:93:4d:ea:fe:49:d9:2d:ab:31.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added '[git.evobot.cn]:8022,[192.54.103.130]:8022' (ECDSA) to the list of known hosts.
  warning: 您似乎克隆了一个空版本库。
  
  ```

- 提示克隆的是一个空版本库，接着可以按照gitlab提示，创建README.md文件并提交：

  ```bash
  [root@localhost ~]# cd java_project/
  [root@localhost java_project]# touch README.md
  [root@localhost java_project]# echo 'Jenkins java deployment repo' > README.md
  [root@localhost java_project]# git add .
  [root@localhost java_project]# git commit -m "add README"
  [master（根提交） 6818812] add README
   1 file changed, 1 insertion(+)
   create mode 100644 README.md
  [root@localhost java_project]# git push -u origin master
  Counting objects: 3, done.
  Writing objects: 100% (3/3), 241 bytes | 0 bytes/s, done.
  Total 3 (delta 0), reused 0 (delta 0)
  To git@git.evobot.cn:evobot/java_project.git
   * [new branch]      master -> master
  分支 master 设置为跟踪来自 origin 的远程分支 master。
  
  ```

- 推送到仓库后，可以到gitlab仓库查看推送的README文件：

![jenkins_java_gitlab4](https://s2.ax1x.com/2019/05/16/EbotZF.png)

## 下载zrlog源码

- 仓库创建完成后，需要将java的源代码推送到git仓库内去，这里使用zrlog的源代码，zrlog源码下载地址在[github](https://github.com/94fzb/zrlog/archive/master.zip)，将源码下载到本地，并解压到之前拉取的git仓库内：

  ```bash
  [root@localhost java_project]# ls
  bin  common  data  doc  LICENSE  mvnw  mvnw.cmd  pom.xml  README.en-us.md  README.md  service  web
  
  ```

- 将zrlog的源码放入到本地仓库目录后，将所有文件改动push到远程仓库：

  ```bash
  $ git add .
  $ git commit -m "add zrlog source code"
  $ git push
  ```

- push到远程仓库完成：

![jenkins_java_zrlog](https://s2.ax1x.com/2019/05/16/EbTX7D.png)

## 安装tomcat

- 为了运行zrlog，还需要在另一台需要部署的机器上安装jdk+tomcat，安装jdk和tomcat可以查看之前的博文[安装 jdk 和 Tomcat](https://www.evobot.cn/post/af3c490c.html)；

- 安装完tomcat后，还需要为Jenkins配置tomcat的管理员用户，编辑tomcat安装目录下的conf目录，编辑目录下的`tomcat-users.xml`文件，找到最后一行`</tomcat-users>`，在上方插入一下内容：

  ```xml
  <role rolename="admin"/>
  <role rolename="admin-script"/>
  <role rolename="admin-gui"/>
  <role rolename="manager"/>
  <role rolename="manager-gui"/>
  <role rolename="manager-script"/>
  <role rolename="manager-jmx"/>
  <role rolename="manager-status"/>
  <user name="admin" password="123456" roles="admin,admin-gui,admin-script,manager,manager-gui,manager-script,manager-jmx,m
  anager-status"/>
  ```

  这里是定义tomcat的用户角色，并且分配了admin账户和密码，并配置了admin账户的所属的角色权限。

- 配置完成后重启tomcat，然后访问tomcat默认主页，在页面下方有一个`Managing Tomcat`，点击提供的`manager webapp`链接可以进入tomcat的管理页面，如果访问管理页面提示403，那么还需要进入tomcat安装目录下的`webapps/manager/META-INF`目录，编辑`context.xml`，在最下方的`allow`配置中配置允许访问的IP地址，默认只允许127.0.0.1的IP访问，修改如下：

  ```xml
  <Context antiResourceLocking="false" privileged="true" >
    <Valve className="org.apache.catalina.valves.RemoteAddrValve"
           allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1|192.168.139.*" />
    <Manager sessionAttributeValueClassNameFilter="java\.lang\.(?:Boolean|Integer|Long|Number|String)|org\.apache\.catalina\.filters\.CsrfPreventionFilter\$LruCache(?:\$1)?|java\.util\.(?:Linked)?HashMap"/>
  </Context>
  ```

  配置允许访问的IP，使用`|`分割不同的IP，同时IP地址也支持正则匹配。

- 再次重启tomcat后访问管理页面，就会弹出输入账号密码的提示，填入之前在`tomcat-users.xml`中配置的用户名密码即可登录，manager页面的配置主要是为了提供给Jenkins使用tomcat的接口，使得Jenkins能够将代码部署到tomcat中。

## maven安装配置

- maven能够帮助我们编译和打包java的源码，[下载maven的二进制包](http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.1/binaries/apache-maven-3.6.1-bin.tar.gz)到Jenkins所在的主机上，然后解压并移动到`/usr/local`目录下；

- 查看maven的版本号，使用命令`/usr/local/apache-maven-3.6.1/bin/mvn --version`:

  ```bash
  [root@localhost apache-maven-3.6.1]# /usr/local/apache-maven-3.6.1/bin/mvn --version
  Apache Maven 3.6.1 (d66c9c0b3152b2e69ee9bac180bb8fcc8e6af555; 2019-04-05T03:00:29+08:00)
  Maven home: /usr/local/apache-maven-3.6.1
  Java version: 1.8.0_212, vendor: Oracle Corporation, runtime: /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.212.b04-0.el7_6.x86_64/jre
  Default locale: zh_CN, platform encoding: UTF-8
  OS name: "linux", version: "3.10.0-862.el7.x86_64", arch: "amd64", family: "unix"
  
  ```

- 登录Jenkins，点击 系统管理 > 全局工具配置，在Maven配置项中，将 **默认 settings 提供** 和  **默认全局 settings 提供** 选为**文件系统中的全局 settings 文件**，而文件路径则填入maven的配置文件路径`/usr/local/apache-maven-3.6.1/conf/settings.xml`；

![jenkins_maven](https://s2.ax1x.com/2019/05/18/EOfd5d.png)

- 然后在页面中找到Maven选项，点击 新增Maven，填入自定义的Name，取消勾选自动安装，填入MAVEN_HOME即maven的安装路径：

![jenkins_maven](https://s2.ax1x.com/2019/05/18/EOfP4s.png)

- 配置完成后保存即可。

## Jenkins安装插件

- 在Jenkins的系统管理 > 插件管理中，检查是否已经安装**Maven Integration plugin**和**Deploy to container Plugin**这两个插件；
- 其中**Maven Integration plugin**插件是用来创建maven项目使用的，而**Deploy to container Plugin**插件则是用来将war包发布到远程主机中去；
- 在插件管理中，点击可选插件，然后找到 **Maven Integration**插件，勾选后点击直接安装，安装后再找到**Deploy to container**插件安装，安装完成后重启Jenkins；
- 重启完成后，登录Jenkins后，点击 新建任务，就会在界面中看到有了**构建一个maven项目**选项：

![jenkins_maven3](https://s2.ax1x.com/2019/05/18/EO5CUe.png)

## 构建job

- 选择构建一个maven项目，自定义任务名称，点击确定；

- 然后定义任务具体设置，在源码管理中选择Git，填入之前创建的zrlog项目的git仓库地址，需要注意的是，如果远程gitlab仓库的ssh端口不是默认的22端口，那么需要在`/var/lib/jenkins/.ssh`目录下添加`config`配置，写入以下内容：

  ```ini
  Host git.evobot.cn
  User git
  Port 8022
  
  ```

![jenkins_maven4](https://s2.ax1x.com/2019/05/18/EO50PJ.png)

- 在下面的 **Credentials** 选项中点击 **添加** > **Jenkins** ， 在弹出的页面中，类型选择 SSH Username with private key，Username填入git，Private Key选择Enter directly，将之前在gitlab上添加的公钥对应的私钥内容填入输入框内，完成后点击添加：

![jenkins_maven5](https://s2.ax1x.com/2019/05/18/EOIKL6.png)

- 添加完成后，在**Credentials**中选择刚才添加的git用户配置即可，Jenkins会自动尝试连接仓库，如果能够章程访问，则仓库地址下的红色提示信息会自动消失；
- 下面的 构建触发器，构建环境，Pre Step三个选项都保持默认；
- 而Build选项，则是maven构建java项目的配置，其中Root POM保持默认，Goals and options则填写`clean install -D maven.test.skip=true`，表示跳过maven构建测试，也可以保持默认留空；
- Post Step和构建设置保持默认配置；
- 构建后操作，先选择 Editable Email Notification，配置好收件人地址，然后保存后，点击立即构建，查看maven是否能够构建完成zrlog的源码：

![jenkins_maven6](https://s2.ax1x.com/2019/05/18/EOqnQH.png)

- 可以看到构建过程中出现了失败的情况，提示是jdk环境存在问题，所以还需要在Jenkins主机上安装jdk，并在 系统管理 > 全局工具配置 中配置 JDK：

![jenkins_jdk](https://s2.ax1x.com/2019/05/18/EOqotK.png)

- 回到任务界面，重新构建任务，构建成功后，会在控制台输出的war包的存放路径：

![jenkins_jdk2](https://s2.ax1x.com/2019/05/18/EOLtN6.png)

## 发布war包

- 通过上面的构建，说明maven的没有问题，那么剩下的就是将编译好的war包部署到远程机器上；
- 点击任务名称进入任务详情，点击配置，进入配置页，在 构建后操作 选项中，点击 增加构建后操作步骤，选择**Deploy war/ear to a container**选项：

![jenkins_publish](https://s2.ax1x.com/2019/05/18/EOOP56.png)

- 在配置中的 **WAR/EAR files** 中填入`**/*.war`，表示所有的war包，在**Containers** 选项中选择tomcat版本：

![jenkins_publish2](https://s2.ax1x.com/2019/05/18/EOOlPf.png)

- 然后在**Credentials**中增加tomcat之前管理页面添加的admin账号和密码：

![jenkins_publish3](https://s2.ax1x.com/2019/05/18/EOO3RS.png)

- 然后**Tomcat URL** 填入tomcat的访问地址，如果tomcat非80端口，则需要增加端口：

![jenkins_publish4](https://s2.ax1x.com/2019/05/18/EOOdI0.png)

- 保存任务，再次构建任务，成功后可以通过 tomcat访问到构建成功的zrlog页面。

---

