---
title: Jumpserver配置与使用
author: Evobot
date: 2018-09-17 20:11:47
categories: 堡垒机
tags:
  - Jumpserver
image:
---

1. 创建jumpserver普通用户
2. 添加机器
3. 添加系统用户并授权
4. 添加授权规则
5. 客户端登录jumpserver

---

# 创建Jumpserver普通用户

- 如果Jumpserver停止运行，可以在Jumpserver目录内，执行`./service.sh restart`重启服务；

- Jumpserver普通用户，是用来登录Jumpserver网页或堡垒机的用户；

- 首先创建用户组，在web界面点击用户管理->查看用户组，在右侧界面点击**添加用户组**，填入用户组名，确认保存即可：

  ![iZrNnA.png](https://s1.ax1x.com/2018/09/17/iZrNnA.png)

- 然后点击用户管理->查看用户，在右侧界面点击**添加用户**，填入用户名，用户名就是登录用户名，然后填入用户姓名，用户组默认选择了之前创建的运维组，权限设置为普通用户，填入用户的邮箱地址，Jumpserver会讲用户的密码、秘钥发送到该邮箱，点击确认保存完成用户添加：

  ![iZr6Xj.png](https://s1.ax1x.com/2018/09/17/iZr6Xj.png)

- 添加用户成功后，Jumpserver会给用户的邮箱发送一封邮件，内容包含web登录密码、ssh秘钥文件密码以及密钥下载地址：

  ![iZr4hT.png](https://s1.ax1x.com/2018/09/17/iZr4hT.png)

- 如果由于邮箱出问题或其他原因导致邮件发送失败，我们也可以在用户管理中的查看用户菜单中为用户重新设置密码：

  ![iZrbu9.png](https://s1.ax1x.com/2018/09/17/iZrbu9.png)

- 而秘钥密码则可以让用户先登录web下载密钥，下载之后，密钥栏将变为**NoKey GenOne?**，这时就可以重新生成密钥，并会显示新的密钥密码：

  ![iZsV4f.png](https://s1.ax1x.com/2018/09/17/iZsV4f.png)

# 添加资产

- 首先点击**资产管理**->**查看资产组**，在右侧页面点击添加主机组，填入主机组名，提交即可；

- 然后点击**资产管理**->**查看资产**，在右侧页面点击**添加资产**，填入主机名、主机IP、端口号，选择所属主机组（按ctrl可以多选），选中激活，然后提交即可完成创建：

  ![iZsIat.png](https://s1.ax1x.com/2018/09/17/iZsIat.png)

- 创建完成后，可以在**查看资产**菜单内，看到新添加的主机，勾选主机，点击更新，Jumpserver可以抓取到主机的操作系统、CPU核数、内存硬盘等信息，这个信息就是使用之前创建管理用户进行获取的，但由于之前在创建管理用户时，并没有给用户sudo权限，所以还需要到客户机上对用户进行设置，使用`visudo`对jump用户进行配置：

  ```bash
  //visudo
  ## Same thing without a password
  # %wheel        ALL=(ALL)       NOPASSWD: ALL
  jump    ALL=(ALL)       NOPASSWD: ALL
  
  ```

  - 然后选中主机，点击更新：

  ![iZyEL9.png](https://s1.ax1x.com/2018/09/17/iZyEL9.png)

  

# 添加系统用户并授权

- 系统用户是跳板机用来登录到客户机的用户，点击**授权管理**->**系统用户**，在右侧菜单中点击添加系统用户；

- 由于我们是为普通用户zhangsan进行的配置，所以这里用户名称依然使用zhangsan，然后填入自定义的密码，密钥建议到Jumpserver服务器后台手动生成，然后将私钥填入输入框：

  ```bash
  cd .ssh
  ssh-keygen -f zhangsan
  cat zhangsan
  ```

- 最后点击确认保存完成系统用户创建，系统用户不需要到客户机上手动创建，Jumpserver可以使用管理用户推送到客户机上进行自动创建，并且会自动生成公钥；

- 在系统用户列表，点击**推送**按钮，会进入系统用户推送页面，在这个页面选择要推送的资产或资产组，提交后，Jumpserver就会将这个系统用户推送到制定的主机上去：

  ![iZ6kff.png](https://s1.ax1x.com/2018/09/17/iZ6kff.png)

  ![iZ6K7n.png](https://s1.ax1x.com/2018/09/17/iZ6K7n.png)

- 我们可以到客户机上查看已经被推送过来的系统用户：

  ```bash
  [jump@localhost .ssh]$ id zhangsan
  uid=1003(zhangsan) gid=1003(zhangsan) 组=1003(zhangsan)
  
  ```

# 授权规则

- 在**授权管理**->**授权规则**菜单中，点击**添加规则**为zhangsan用户添加授权规则；

- 在添加规则页面中填入自定义的授权名称、选择需要授权的用户或者用户组，然后选择需要授权给用户的资产主机或资产组，并且选择需要关联的系统用户，确认保存之后完成添加规则，*需要注意的是，如果系统用户未推送到对应的资产或资产组，那么保存时会提示主机未推送选择的系统用户*：

  ![iZcDVs.png](https://s1.ax1x.com/2018/09/17/iZcDVs.png)

- 授权规则，作用是产生一个映射，让Jumpserver里的用户与指定资产里的机器上的系统用户产生关联；

- Jumpserver的用户关系如下图：

  ![iZcbRK.png](https://s1.ax1x.com/2018/09/17/iZcbRK.png)

  - Jumpserver的普通用户用来登录Jumpserver的web界面或者进行ssh登录Jumpserver服务器；
  - 而管理用户则用来自动创建客户机上的系统用户、批量执行命令等操作；
  - 客户机上的系统用户，是Jumpserver用来登录每个客户机的用户；
  - 所以授权规则就是将Jumpserver的普通用户与客户机上的系统用户关联起来，让普通用户有权限对授权的客户机进行操作。

# 客户端登录Jumpserver

- 在添加完规则后，我们可以使用ssh客户端用普通用户连接堡垒机，连接使用秘钥认证，秘钥就是在创建普通用户时，Jumpserver自动生成让用户下载的私钥；

- 实际上我们在Jumpserver上创建的普通用户，在系统上可以看到其对应的bash并不是`/bin/bash`,而是下面这种：

  ```bash
  [root@vm1 .ssh]# cat /etc/passwd | grep zhangsan
  zhangsan:x:1006:1006::/home/zhangsan:/root/jumpserver/init.sh
  
  ```

- 所以用户使用密钥认证登录堡垒机后，显示的登录界面如下：

  ```bash
  ###    欢迎使用Jumpserver开源跳板机系统   ### 
  
          1) 输入 ID 直接登录.
          2) 输入 / + IP, 主机名 or 备注 搜索.
          3) 输入 P/p 显示您有权限的主机.
          4) 输入 G/g 显示您有权限的主机组.
          5) 输入 G/g + 组ID 显示该组下主机.
          6) 输入 E/e 批量执行命令.
          7) 输入 U/u 批量上传文件.
          8) 输入 D/d 批量下载文件.
          9) 输入 H/h 帮助.
          0) 输入 Q/q 退出.
  
  Opt or ID>: 
  
  ```

- 根据登录显示的提示，我们可以查看当前登录用户有权限登录的客户机有哪些：

  ```bash
  Opt or ID>: p
  [ID ] 主机名    IP               端口  系统用户  备注
  [0  ] vm2             192.168.49.129   22     ['zhangsan']  
  
  ```

- 使用ID或者主机名，都可以直接登录到客户机上去；

- 按e可以进入批量执行命令界面，然后可以输入需要批量执行命令的主机名，然后直接输入需要执行的命令，就可以在制定的多台机器上进行执行并输出执行结果了，执行完后需要输入q退出。

—