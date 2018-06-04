---
title: Windows10开机自启动Linux子系统和ssh服务
author: Evobot
abbrlink: 5a8b1c0f
date: 2018-06-01 17:17:21
categories: Linux
tags: [windows10,WSL]
image:
---

win10中的Linux子系统默认无法开机自启动，并且ssh服务也需要每次启动bash后手动启动，这里使用两个脚本来让Linux子系统在系统启动时也自行启动，并且将ssh服务打开。

<!--more-->

---

# win10配置

- 创建一个批处理脚本`wslstartup.bat`，写入如下内容：

  ```powershell
  powershell.exe -WindowStyle Hidden -c "bash /init.sh "
  ```

  - 这里表示隐藏窗口启动Linux子系统，并执行`/init.sh`shell脚本。

- 打开运行，输入`shell:startup`回车，打开windows启动文件夹，将创建的批处理脚本移动进去。

# Linux子系统配置
- 在根目录创建`init.sh`shell脚本，写入以下内容：
  ```bash
  #!/bin/bash
  pn=$(ps aux | grep -v grep |grep sshd|wc -l)
  if [ "${pn}" != "0" ]; then
      pid=$(ps aux|grep -v grep|grep /usr/sbin/sshd|awk '{print $2}')
      echo "123456" | sudo -S kill $pid
  fi
  echo "123456" | sudo -S /usr/sbin/service ssh start
  ```
  - 其中`echo`的内容为默认登陆用户的登陆密码。
- 更改脚本权限，和更改属主和属组为默认用户：
  ```bash
  # chmod 755 /init.sh
  # chown user:user /init.sh
  ```
- 配置完成后，下次开机就可以自启动Linux子系统，并且将ssh服务启动，我们就可以在xshell等软件中登陆子系统了。
---

