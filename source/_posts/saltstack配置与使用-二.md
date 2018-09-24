---
title: saltstack配置与使用(二)
author: Evobot
date: 2018-09-20 19:08:27
categories: 自动化运维
tags:
  - saltstack
image:
---

saltstack配置管理目录、配置管理远程命令，配置管理计划任务；介绍saltstack其他命令的使用方法和salt-ssh的使用方法。

<!--more-->

---

# saltstack管理功能

## 配置管理目录

- saltstack除了进行文件分发之外，同样也可以对目录进行管理，将目录分发到指定的主机上去；

- 首先进入`/srv/salt/`目录，创建一个`test_dir.sls`配置文件，写入以下配置内容：

  ```bash
  file_dir:	#配置名
    file.recurse:	#saltstack模块，用来传输目录
      - name: /tmp/testdir	#客户端上目录的路径
      - source: salt://test/123	#源目录路径
      - user: root	#属主
      - file_mode: 640	#文件权限
      - dir_mode: 750	#目录权限
      - mkdir: True	#如果客户端路径不存在是否创建目录
      - clean: True	#表示源删除文件或目录后，目标主机上也会进行删除，否则不删除
  
  ```

- 保存之后，修改`top.sls`：

  ```bash
  base:
    '*':
      - test	#即使test配置已经执行过，这里不删除也不会影响
      - test_dir
  
  ```

- 然后执行`salt ‘hostname’ stat.highstate`命令进行目录的自动分发：

  ```bash
  [root@vm1 salt]# salt 'vm2' state.highstate
  vm2:
  ----------
            ID: file_test
      Function: file.managed
          Name: /tmp/evobot.conf
        Result: True
       Comment: File /tmp/evobot.conf is in the correct state
       Started: 20:06:04.796103
      Duration: 232.444 ms
       Changes:   
  ----------
            ID: file_dir
      Function: file.recurse
          Name: /tmp/testdir
        Result: True
       Comment: Recursively updated /tmp/testdir
       Started: 20:06:05.028873
      Duration: 210.165 ms
       Changes:   
                ----------
                /tmp/testdir/1.txt:
                    ----------
                    diff:
                        New file
                    mode:
                        0640
  
  Summary for vm2
  ------------
  Succeeded: 2 (changed=1)
  Failed:    0
  ------------
  Total states run:     2
  Total run time: 442.609 ms
  
  ```

- 然后可以在客户端上查看目录和文件是否被分发到主机上：

  ```bash
  [root@vm2 ~]# ls -l /tmp/testdir/1.txt 
  -rw-r-----. 1 root root 511 9月  20 20:06 /tmp/testdir/1.txt
  [root@vm2 ~]# ls -ld /tmp/testdir/ 
  drwxr-x---. 2 root root 19 9月  20 20:06 /tmp/testdir/
  ```

- 接下来我们在master上的`/srv/salt/test/123`目录下创建一个空目录，然后再进行分发：

  ```bash
  [root@vm1 test]# cd /srv/salt/test/123/
  [root@vm1 123]# mkdir abc
  [root@vm1 123]# salt 'vm2' state.highstate
  vm2:
  ----------
            ID: file_test
      Function: file.managed
          Name: /tmp/evobot.conf
        Result: True
       Comment: File /tmp/evobot.conf is in the correct state
       Started: 21:44:24.913794
      Duration: 152.474 ms
       Changes:   
  ----------
            ID: file_dir
      Function: file.recurse
          Name: /tmp/testdir
        Result: True
       Comment: The directory /tmp/testdir is in the correct state
       Started: 21:44:25.066603
      Duration: 70.618 ms
       Changes:   
  
  Summary for vm2
  ------------
  Succeeded: 2
  Failed:    0
  ------------
  Total states run:     2
  Total run time: 223.092 ms
  
  ```

- 再到vm2上查看：

  ```bash
  [root@vm2 tmp]# tree testdir/
  testdir/
  └── 1.txt
  
  0 directories, 1 file
  
  ```

  - 这里会发现abc目录并没有同步，这是因为saltstack**不会同步空目录**，要解决这个问题，就要在目录下创建一个空文件再分发即可：

  ```bash
  [root@vm1 123]# salt 'vm2' state.highstate
  vm2:
  ----------
            ID: file_test
      Function: file.managed
          Name: /tmp/evobot.conf
        Result: True
       Comment: File /tmp/evobot.conf is in the correct state
       Started: 21:48:42.867255
      Duration: 670.436 ms
       Changes:   
  ----------
            ID: file_dir
      Function: file.recurse
          Name: /tmp/testdir
        Result: True
       Comment: Recursively updated /tmp/testdir
       Started: 21:48:43.538084
      Duration: 153.028 ms
       Changes:   
                ----------
                /tmp/testdir/abc:
                    ----------
                    /tmp/testdir/abc:
                        New Dir
                /tmp/testdir/abc/empty.txt:
                    ----------
                    diff:
                        New file
                    mode:
                        0640
  
  Summary for vm2
  ------------
  Succeeded: 2 (changed=1)
  Failed:    0
  ------------
  Total states run:     2
  Total run time: 823.464 ms
  
  ```

  - vm2上查看

  ```bash
  [root@vm2 tmp]# tree testdir/
  testdir/
  ├── 1.txt
  └── abc
      └── empty.txt
  
  1 directory, 2 files
  
  ```

- 另外需要注意，我们这里test_dir.sls中定义的源目录是`/test/123`目录，所以再同步的时候，`/test/123`这两个目录是必须存在的。

## 配置管理远程命令

- 我们使用了`salt ‘hostname’ cmd.run “cmd”`命令来执行远程命令，但实际上，saltstack是支持通过配置来管理多个远程命令分发的；

- 与之前分发文件和目录相同，在master上的`/srv/salt/`目录下创建`shell_test.sls`文件，写入以下配置：

  ```bash
  shell_test:
    cmd.script:
      - source: salt://test/1.sh	#将在minion上执行的shell脚本
      - user: root
  
  ```

- 接着创建`/test/1.sh`脚本文件，写入要执行的命令：

  ```bash
  #!/bin/bash
  touch /tmp/111.txt
  if [ ! -d /tmp/1233 ];then
      mkdir /tmp/1233
  fi
  ```

- 然后在top.sls文件中添加配置文件名：

  ```bash
  base:
    '*':
      - shell_test
  
  ```

- 最后执行分发：

  ```bash
  [root@vm1 salt]# salt 'vm2' state.highstate
  vm2:
  ----------
            ID: shell_test
      Function: cmd.script
        Result: True
       Comment: Command 'shell_test' run
       Started: 22:01:57.004778
      Duration: 198.983 ms
       Changes:   
                ----------
                pid:
                    2899
                retcode:
                    0
                stderr:
                stdout:
  
  Summary for vm2
  ------------
  Succeeded: 1 (changed=1)
  Failed:    0
  ------------
  Total states run:     1
  Total run time: 198.983 ms
  
  ```

- 到vm2上查看是否执行成功：

  ```bash
  [root@vm2 tmp]# ls -l
  总用量 4
  -rw-r--r--. 1 root   root     0 9月  20 22:01 111.txt
  drwxr-xr-x. 2 root   root     6 9月  20 22:01 1233
  
  ```

## 配置管理计划任务

- 这里介绍怎么配置saltstack管理计划任务，与上面的分发文件、目录、执行命令类似，首先在`/srv/salt/`目录下创建`cron_test.sls`文件，写入配置如下：

  ```bash
  cron_test:
    cron.present:	# saltstack模块，表示添加crontab任务
      - name: /bin/touch /tmp/111.txt
      - user: root
      - minute: '*'	#这里的*需要使用单引号括起来
      - hour: 20
      - daymonth: '*'
      - month: '*'
      - dayweek: '*'
  ```

  - 这里配置的意思就是每天20点的每一分钟都执行`touch /tmp/111.txt`命令

- 如果写成下面的形式，使用`cron.absent`模块，则表示删除一个crontab任务，使用`cron.absent`时，不能再使用`cron.present`模块，两者不能共存：

  ```bash
  cron_del:
    cron.absent:
      - name: /bin/touch /tmp/111.txt
  ```

- 修改top.sls配置，添加cron_tab:

  ```bash
  base:
    '*':
      - cron_test
  
  ```

- 执行分发命令：

  ```bash
  [root@vm1 salt]# salt 'vm2' state.highstate
  vm2:
  ----------
            ID: cron_test
      Function: cron.present
          Name: /bin/touch /tmp/111.txt
        Result: True
       Comment: Cron /bin/touch /tmp/111.txt added to root's crontab
       Started: 22:16:16.863887
      Duration: 385.968 ms
       Changes:   
                ----------
                root:
                    /bin/touch /tmp/111.txt
  
  Summary for vm2
  ------------
  Succeeded: 1 (changed=1)
  Failed:    0
  ------------
  Total states run:     1
  Total run time: 385.968 ms
  
  ```

- 到vm2上查看crontab任务：

  ```bash
  [root@vm2 tmp]# crontab -l
  # Lines below here are managed by Salt, do not edit
  # SALT_CRON_IDENTIFIER:/bin/touch /tmp/111.txt
  * 20 * * * /bin/touch /tmp/111.txt
  
  ```

  - **需要注意，在客户端主机上crontab任务中，saltstack会自动添加两行注释掉的内容，这些内容不能修改，否则saltstack就无法再对这条cron进行管理**

- 删除一个crontab任务：

  ```bash
  //cron_test.sls
  cron_test:
    cron.absent:
      - name: /bin/touch /tmp/111.txt
  ```

  ```bash
  //执行分发
  [root@vm1 salt]# salt 'vm2' state.highstate
  vm2:
  ----------
            ID: cron_test
      Function: cron.absent
          Name: /bin/touch /tmp/111.txt
        Result: True
       Comment: Cron /bin/touch /tmp/111.txt removed from root's crontab
       Started: 22:20:22.570352
      Duration: 417.0 ms
       Changes:   
                ----------
                root:
                    /bin/touch /tmp/111.txt
  
  Summary for vm2
  
  ```

  ```bash
  //查看vm2的crontab
  [root@vm2 tmp]# crontab -l
  # Lines below here are managed by Salt, do not edit
  
  ```

# saltstack其他命令

- `salt 'hostname' cp.get_file salt://dir/filename /miniondir/filename` ：拷贝master上的文件到客户端：

  ```bash
  [root@vm1 salt]# salt 'vm2' cp.get_file salt://test/1.sh /tmp/vm1.sh
  vm2:
      /tmp/vm1.sh
  
  [root@vm2 tmp]# ls -l vm1.sh 
  -rw-r--r--. 1 root root 82 9月  20 22:24 vm1.sh
  
  ```

- `salt ’hostname' cp.get_dir salt://dir/ /minion/dir`：拷贝目录到minion指定目录下：

  ```bash
  [root@vm1 salt]# mkdir test/vm1dir
  [root@vm1 salt]# touch test/vm1dir/xxx	# 目录下必须有文件，否则saltstack不会同步空目录
  [root@vm1 salt]# salt 'vm2' cp.get_dir salt://test/vm1dir /tmp/
  vm2:
      - /tmp/]/vm1dir/xxx	# saltstack会自动在minion指定目录下创建目录
  
  [root@vm2 tmp]# ls -l vm1dir/xxx
  -rw-r--r--. 1 root root 0 9月  20 22:33 vm1dir/xxx
  
  ```

- `salt-run manage.up` ：显示存活的minion：

  ```bash
  [root@vm1 salt]# salt-run manage.up
  - vm1
  - vm2
  
  ```

- `salt ‘*’ cmd.script salt://dir/script.sh`：命令行下执行master上的shell脚本：

  ```bash
  [root@vm1 salt]# salt 'vm2' cmd.script salt://test/1.sh
  vm2:
      ----------
      pid:
          3387
      retcode:
          0
      stderr:
      stdout:
  
  ```

# salt-ssh使用

- salt-ssh不需要对客户端做认证，客户端也不需要安装salt-minion，它类似pssh/expect；

- 使用salt-ssh需要先安装`salt-ssh`软件包；然后编辑`/etc/salt/roster`配置文件，增加以下内容：

  ```bash
  vm2:	#主机名
    host: 192.168.49.129	# IP地址
    user: root	#登录的用户名，如果非root，则需要给用户配置sudo权限
    passwd: 123456
  
  ```

- 使用`salt-ssh --key-deploy ‘*’ -r ‘w’`命令讲本机的公钥推送到上面配置的主机上去，然后就可以在配置文件中删除密码，这里salt-ssh推送的公钥是其自己生成的公钥，位于`/etc/salt/pki/master/ssh/`目录下：

  ```bash
  [root@vm1 salt]# salt-ssh --key-deploy '*' -r 'w'
  vm2:
      ----------
      retcode:
          0
      stderr:
      stdout:
          root@192.168.49.129's password: 
           23:04:34 up  3:01,  2 users,  load average: 0.00, 0.01, 0.05
          USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
          root     tty1                      20:03    3:00m  0.02s  0.02s -bash
          root     pts/0    192.168.49.1     20:03   10.00s  0.09s  0.09s -bash
  
  ```

  - 这里需要注意，如果是第一次连接主机，ssh登录会提示输入yes/no，这会导致salt-ssh推送公钥失败，所以在执行推送公钥命令时，要先手动使用ssh登录一次主机。

- 公钥推送成功后，就可以删除配置文件中配置的root密码了：

  ```bash
  vm2:
    host: 192.168.49.129
    user: root
  
  ```

  - 再执行salt-ssh，即可无密码登录：

  ```bash
  [root@vm1 salt]# salt-ssh --key-deploy '*' -r 'w'
  vm2:
      ----------
      retcode:
          0
      stderr:
      stdout:
           23:12:44 up  3:09,  2 users,  load average: 0.00, 0.01, 0.05
          USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
          root     tty1                      20:03    3:09m  0.02s  0.02s -bash
          root     pts/0    192.168.49.1     20:03    7:48   0.09s  0.09s -bash
  
  ```
---

