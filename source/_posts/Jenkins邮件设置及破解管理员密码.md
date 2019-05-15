---
title: Jenkins邮件设置及破解管理员密码
author: Evobot
date: 2019-05-14 22:45:02
categories: 持续集成
tags: Jenkins
image:
---



1. jenkins邮件设置
2.  插件email-ext
3.  破解jenkins管理员密码

---

# Jenkins配置邮件

## 邮箱配置

- 在Jenkins中配置了任务之后，每次构建任务，无论成功或者失败，都应该通知到相应的人，在Jenkins中内置的有邮件系统;
- 在Jenkins的 系统管理 > 系统设置 >  Jenkins Location 的系统管理员邮件地址输入框中，输入管理员的邮箱地址，这个地址需要保证与下面的 邮件通知 配置中的SMTP认证的用户名保持一致；

![jenkins_email](https://s2.ax1x.com/2019/05/14/EobiLD.png)

- 然后在下面的 邮件通知 配置中，首先填入 SMTP服务器 地址，然后勾选 使用SMTP认证，填入邮箱的用户名、密码和SMTP端口：

![jenkins_email2](https://s2.ax1x.com/2019/05/14/EoqgEQ.png)

- 接着勾选 通过发送测试邮件测试配置，填入要收件人邮箱地址，点击 Test configuration，成功发送邮件，会提示 Email was successfully sent ：

![jenkins_email3](https://s2.ax1x.com/2019/05/14/EoLJx0.png)

## 任务邮件配置

- 在系统设置中配置好邮箱之后，还需要在已经创建的任务中修改配置，点击任务名称后面的下拉箭头，选择 配置，在弹出的任务配置中，找到 构建后操作 选项，点击 增加构建后操作步骤：

![jenkins_email4](https://s2.ax1x.com/2019/05/14/EoXR4H.png)

![jenkins_email5](https://s2.ax1x.com/2019/05/14/EoXfCd.png)

- 然后在 Recipients 中填入收件人的地址，勾选下面的 每次不稳定的构建都发送邮件通知，然后点击应用和保存，保存配置：

![jenkins_email6](https://s2.ax1x.com/2019/05/14/EoXqUg.png)

- 接着我们修改任务，模拟构建失败的场景，看邮件是否能够成功发送：

![jenkins_email7](https://s2.ax1x.com/2019/05/14/EojEG9.png)

- 查看邮箱收到的邮件：

![jenkins_emial8](https://s2.ax1x.com/2019/05/14/EojZx1.png)

- 上面是用Jenkins自带的邮件通知系统进行邮件通知，但这个邮件通知只能在任务构建不稳定或者失败的时候给收件人发送邮件，如果我们想要不论构建成功还是失败，都发送邮件通知，那么就需要使用 email-ext插件来实现这样的功能。

## email-ext插件

- 插件名字为Email Extension Plugin，默认是已经安装了的，可以到 系统管理 > 插件管理 > 已安装 菜单内查看是否安装：

![jenkins_email_ext](https://s2.ax1x.com/2019/05/15/E7ftSS.png)

- 点击 系统管理 > 系统设置 > Extended E-mail Notification，使用email-ext之前，需要将上面配置的Jenkins自带的邮件配置取消；
- 首先在 SMTP server 中填入smtp服务器地址，然后点击右下方的 高级 按钮，勾选 Use SMTP Authentication，填写用户名、密码和SMTP port：

![jenins_email_ext2](https://s2.ax1x.com/2019/05/15/E7hStP.png)

- 其余配置保持默认，下拉到 Extended E-mail Notification 配置的最下方，点击右下角的 Default Triggers 按钮，弹出的选项是指定什么情况下发送邮件，我们选择 Always：

![jenkins_eamil_ext3](https://s2.ax1x.com/2019/05/15/E7hYA1.png)

- 配置完成后，点击应用、保存，然后修改任务配置，删除之前配置的构建后操作中的E-mail Notification，点击 增加构建后操作步骤，选择 Editable Email Notification（可编辑的email通知） 选项：

![jenkins_email_ext4](https://s2.ax1x.com/2019/05/15/E747se.png)

- 该选项调用的就是 Extended E-mail Notification 插件，在弹出的选项中，首先修改 Project Recipient List （项目接收人列表）选项，填入收件人邮箱地址，使用`,`分割：

![jenkins_email_ext5](https://s2.ax1x.com/2019/05/15/E75ZWV.png)

- 然后下拉到最后点击 Advanced Setting 按钮，在 Triggers （触发器）选项中，已经自动添加了 Always 触发器，即总是发送邮件，我们也可以自行添加新的触发器，点击 Add Trigger 按钮，在下拉列表内可以选择 Success，构建成功时发送邮件：

![jenkins_email_ext6](https://s2.ax1x.com/2019/05/15/E75weH.png)

- 在 Editable Email Notification 配置中，我们可以自定义邮件的回复列表、内容类型、默认邮件主题、默认邮件正文、发送前是否要调用脚本等配置，其中 Attach Build Log 可以将构建的日志作为附件发送给收件人：

![jenkins_email_ext7](https://s2.ax1x.com/2019/05/15/E75ROg.png)

- 配置完成后，保存任务配置，发起构建，测试构建成功后是否发送邮件：

![jenkins_email_ext8](https://s2.ax1x.com/2019/05/15/E75j0J.png)

- 更多关于Jenkins的邮件配置，可以参考[博文](http://www.cnblogs.com/zz0412/p/jenkins_jj_01.html)。

# Jenkins破解管理员密码

- 对于丢失了管理员密码的情况，需要我们破解管理员密码；

- 首先进入Jenkins的管理员用户目录：

  ```bash
  cd /var/lib/jenkins/users/admin_406465659830194984/
  ```

- 然后编辑目录下的`config.xml`，在文件中找到`passwordHash`字段，这个字段存储的就是管理员用户的密码哈希值，然后使用下面的配置替换该字段内容：

  ```xml
  <passwordHash>#jbcrypt:$2a$10$sO/I.LHeyOfCEA1DpMVSlOgxu8rmpgixjQeMMw7BkPKbKR1LK1lpi</passwordHash>
  ```

- 重启Jenkins服务，然后重新登录，新密码为123456。

---

