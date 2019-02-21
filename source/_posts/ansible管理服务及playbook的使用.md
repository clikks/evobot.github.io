---
title: ansible管理服务及playbook的使用
author: Evobot
categories: 自动化运维
tags:
  - ansible
abbrlink: 54a1f6fc
date: 2018-09-25 22:44:19
image:
---

1. ansible安装包和管理服务
2.  使用ansible playbook
3. playbook里的变量
4. playbook里的循环
5. playbook里的条件判断
6. playbook中的handlers

<!--more-->

---

# ansible安装软件及管理服务

## 安装软件

- ansible远程安装软件使用`yum`模块，命令格式为`ansible [group] -m yum -a "name=[rpmname]"`：

  ```bash
  [root@vm1 ~]# ansible vm2 -m yum -a "name=zsh"
  vm2 | SUCCESS => {
      "changed": true, 
      "msg": "", 
      "rc": 0, 
      "results": [
          "Loaded plugins: fastestmirror\nLoading mirror speeds from cached hostfile\n * base: mirrors.163.com\n * extras: mirrors.aliyun.com\n * updates: mirrors.aliyun.com\nResolving Dependencies\n--> Running transaction check\n---> Package zsh.x86_64 0:5.0.2-28.el7 will be installed\n--> Finished Dependency Resolution\n\nDependencies Resolved\n\n================================================================================\n Package        Arch              Version                 Repository       Size\n================================================================================\nInstalling:\n zsh            x86_64            5.0.2-28.el7            base            2.4 M\n\nTransaction Summary\n================================================================================\nInstall  1 Package\n\nTotal download size: 2.4 M\nInstalled size: 5.6 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  Installing : zsh-5.0.2-28.el7.x86_64                                      1/1 \n  Verifying  : zsh-5.0.2-28.el7.x86_64                                      1/1 \n\nInstalled:\n  zsh.x86_64 0:5.0.2-28.el7                                                     \n\nComplete!\n"
      ]
  }
  
  ```

- 而远程卸载一个软件包，则使用`ansible [group] -m yum -a "name=[rpmname] state=removed"`命令：

  ```bash
  [root@vm1 ~]# ansible vm2 -m yum -a "name=zsh state=removed"
  vm2 | SUCCESS => {
      "changed": true, 
      "msg": "", 
      "rc": 0, 
      "results": [
          "已加载插件：fastestmirror\n正在解决依赖关系\n--> 正在检查事务\n---> 软件包 zsh.x86_64.0.5.0.2-28.el7 将被 删除\n--> 解决依赖关系完成\n\n依赖关系解决\n\n================================================================================\n Package       架构             版本                      源               大小\n================================================================================\n正在删除:\n zsh           x86_64           5.0.2-28.el7              @base           5.6 M\n\n事务概要\n================================================================================\n移除  1 软件包\n\n安装大小：5.6 M\nDownloading packages:\nRunning transaction check\nRunning transaction test\nTransaction test succeeded\nRunning transaction\n  正在删除    : zsh-5.0.2-28.el7.x86_64                                     1/1 \n  验证中      : zsh-5.0.2-28.el7.x86_64                                     1/1 \n\n删除:\n  zsh.x86_64 0:5.0.2-28.el7                                                     \n\n完毕！\n"
      ]
  }
  
  ```

- 可以看到卸载软件使用了`state=removed`参数，其实安装软件的参数也可以加上`state=installed`，但默认不加就是安装软件包，所以这个参数可以省略。

## 管理服务

- ansible远程管理服务使用`service`模块，命令格式为`ansible [group] -m service -a "name=[service_name] state=[started|stopped] enabled=[yes|no]"`,这里的enabled是指定服务是否开机启动：

  ```bash
  [root@vm1 ~]# ansible vm2 -m service -a "name=httpd state=started enabled=no"
  vm2 | SUCCESS => {
      "changed": true, 
      "enabled": false, 
      "name": "httpd", 
      "state": "started", 
      "status": {
          "ActiveEnterTimestampMonotonic": "0", 
          "ActiveExitTimestampMonotonic": "0", 
          "ActiveState": "inactive", 
          "After": "remote-fs.target network.target basic.target -.mount nss-lookup.target systemd-journald.socket tmp.mount system.slice", 
  		...
      }
  }
  
  ```

- 查看vm2上的httpd服务是否启动：

  ```bash
  [root@vm2 ~]# ps aux|grep httpd
  root      18137  0.1  1.0 226264  5180 ?        Ss   23:12   0:00 /usr/sbin/httpd -DFOREGROUND
  apache    18138  0.0  0.6 226264  3012 ?        S    23:12   0:00 /usr/sbin/httpd -DFOREGROUND
  apache    18139  0.0  0.6 226264  3012 ?        S    23:12   0:00 /usr/sbin/httpd -DFOREGROUND
  apache    18140  0.0  0.6 226264  3012 ?        S    23:12   0:00 /usr/sbin/httpd -DFOREGROUND
  apache    18141  0.0  0.6 226264  3012 ?        S    23:12   0:00 /usr/sbin/httpd -DFOREGROUND
  apache    18142  0.0  0.6 226264  3012 ?        S    23:12   0:00 /usr/sbin/httpd -DFOREGROUND
  
  [root@vm2 ~]# systemctl is-enabled httpd
  disabled
  
  ```

## ansible-doc命令

- `ansible-doc`命令可以查看模块的用法，使用这个命令需要安装`ansible-doc`软件包；

- `ansible-doc -l`可以列出所有的模块，`ansible-doc [module_name]`命令则可以查看模块的参数和用法：

  ```bash
  [root@vm1 ~]# ansible-doc -l
  a10_server                                Manage A10 Networks AX/SoftAX/Thunder/vThunder devi...
  a10_server_axapi3                         Manage A10 Networks AX/SoftAX/Thunder/vThunder devi...
  a10_service_group                         Manage A10 Networks AX/SoftAX/Thunder/vThunder devi...
  a10_virtual_server                        Manage A10 Networks AX/SoftAX/Thunder/vThunder devi...
  accelerate                                Enable accelerated mode on remote node             
  aci_aep                                   Manage attachable Access Entity Profile (AEP) on Ci...
  aci_ap                                    Manage top level Application Profile (AP) objects o...
  aci_bd                                    Manage Bridge Domains (BD) on Cisco ACI Fabrics (fv...
  aci_bd_subnet                             Manage Subnets on Cisco ACI fabrics (fv:Subnet)    
  aci_bd_to_l3out                           Bind Bridge Domain to L3 Out on Cisco ACI fabrics (...
  aci_config_rollback                       Provides rollback and rollback preview functionalit...
  aci_config_snapshot                       Manage Config Snapshots on Cisco ACI fabrics (confi...
  ...
  ```

  ```bash
  [root@vm1 ~]# ansible-doc service
  > SERVICE    (/usr/lib/python2.7/site-packages/ansible/modules/system/service.py)
  
          Controls services on remote hosts. Supported init systems include BSD
          init, OpenRC, SysV, Solaris SMF, systemd, upstart. For Windows
          targets, use the [win_service] module instead.
  
    * note: This module has a corresponding action plugin.
  
  OPTIONS (= is mandatory):
  
  - arguments
          Additional arguments provided on the command line
          (Aliases: args)[Default: (null)]
  
  - enabled
          Whether the service should start on boot. *At least one of state and
          enabled are required.*
          (Choices: yes, no)[Default: (null)]
  
  ```

# ansible playbook

## playbook使用

- ansible的playbook相当于将模块写到配置文件里，而不是使用命令行逐条命令执行；

- 在`/etc/ansible`目录下创建`test.yml`，ansible的playbook为`.yml`后缀名，写入的内容如下所示，可以展示出playbook的格式：

  ```yaml
  ---	#第一行必须为三条'-'
  - hosts: vm2	#- hosts指定操作的主机,多个主机使用',’分隔，也可以使用主机组
    remote_user: root	# 指定操作使用的用户身份
    tasks:	# 指定执行的任务
      - name: test_playbook	# playbook的名字，在执行时会打印
        shell: touch /tmp/evobot.txt	#使用的模块为shell，然后为执行的命令
  
  ```

- 保存之后，使用`ansible-playbook [playbook_name.yml]`命令运行playbook：

  ```bash
  [root@vm1 ansible]# ansible-playbook test.yml 
  
  PLAY [vm2] ***************************************************************************************
  
  TASK [Gathering Facts] ***************************************************************************
  ok: [vm2]
  
  TASK [test_playbook] *****************************************************************************
   [WARNING]: Consider using file module with state=touch rather than running touch
  
  changed: [vm2]
  
  PLAY RECAP ***************************************************************************************
  vm2                        : ok=2    changed=1    unreachable=0    failed=0   
  
  
  ```

- 在vm2上查看运行结果：

  ```bash
  [root@vm2 ~]# ls -l /tmp/evobot.txt 
  -rw-r--r--. 1 root root 0 9月  25 23:28 /tmp/evobot.txt
  
  ```

## playbook中的变量

- 重新创建一个playbook，文件名为`create_user.yml`，用来创建用户的playbook，内容如下：

  ```yaml
  ---
  - name: create_user
    hosts: vm2
    user: root
    gather_facts:false
    vars:
      - user: "test"
    tasks:
      - name: create user
        user: name="{{user}}"
  
  ```

- 上面的playbook中，`gather_facts`用来收集客户机上的一些属性信息，这里将其关闭,在机器数量较多时，建议关闭，减轻ansible压力，`vars`用来定义变量，变量名为`user`，值为`"test"`以便在后面进行引用；然后在tasks中使用了`user`模块，并且使用双重花括号引用之前定义的变量`user`。

- 执行结果如下：

  ```bash
  [root@vm1 ansible]# ansible-playbook create_user.yml 
  
  PLAY [create_user] *******************************************************************************
  
  TASK [create user] *******************************************************************************
  changed: [vm2]
  
  PLAY RECAP ***************************************************************************************
  vm2                        : ok=1    changed=1    unreachable=0    failed=0   
  
  [root@vm2 ~]# id test
  uid=1001(test) gid=1001(test) 组=1001(test)
  
  ```

- 如果创建的用户已经存在，在执行playbook时，输出信息的`changed`就是为0。

## playbook循环

- 再创建一个`while.yml`palybook，写入如下内容：

  ```yaml
  ---
  - hosts: vm2
    user: root
    tasks:
      - name: change mode for files
        file: path=/tmp/{{ item }} state=touch  mode=600	
        with_items:
          - 1.txt
          - 2.txt
          - 3.txt
  
  ```

- 上面的playbook使用了file模块，创建文件，并将权限设置为600，其中创建的文件名，使用了变量`{{ item }}`，在下面使用了`with_items`设置循环，这样在执行p'laybook时，就会将`with_items`中定义的三个文件名，传递给变量`item`，并使用`file`模块进行创建，

- 执行结果如下：

  ```bash
  [root@vm1 ansible]# ansible-playbook while.yml 
  
  PLAY [vm2] ***************************************************************************************
  
  TASK [Gathering Facts] ***************************************************************************
  ok: [vm2]
  
  TASK [change mode for files] *********************************************************************
  changed: [vm2] => (item=1.txt)
  changed: [vm2] => (item=2.txt)
  changed: [vm2] => (item=3.txt)
  
  PLAY RECAP ***************************************************************************************
  vm2                        : ok=2    changed=1    unreachable=0    failed=0   
  
  
  ```

- ansible playbook中定义循环是先定义变量，再使用`with_items`进行循环。

## playbook中的条件判断

- 之前使用的`gather_facts`，类似于saltstack中的grains，会对客户机的信息进行收集，而查看这些信息，则使用命令`ansible [hostname] -m setup`，使用了`setup`模块：

  ```bash
  [root@vm1 ansible]# ansible vm2 -m setup
  vm2 | SUCCESS => {
      "ansible_facts": {
          "ansible_all_ipv4_addresses": [
              "192.168.199.142"
          ], 
          "ansible_all_ipv6_addresses": [
              "fe80::3ef1:a048:4d1d:67cd"
          ], 
          "ansible_apparmor": {
              "status": "disabled"
          }, 
          "ansible_ens33": {
              "active": true, 
              "device": "ens33", 
              "features": {
                  "busy_poll": "off [fixed]", 
                  "fcoe_mtu": "off [fixed]", 
                  "generic_receive_offload": "on", 
                  "generic_segmentation_offload": "on", 
                  "highdma": "off [fixed]", 
                  "hw_tc_offload": "off [fixed]", 
                  "l2_fwd_offload": "off [fixed]", 
                  "large_receive_offload": "off [fixed]", 
                  "loopback": "off [fixed]", 
                  "netns_local": "off [fixed]", 
                  "ntuple_filters": "off [fixed]", 
                  "receive_hashing": "off [fixed]", 
                  "rx_all": "off", 
                  "rx_checksumming": "off", 
                  "rx_fcs": "off", 
                  "rx_vlan_filter": "on [fixed]", 
                  "rx_vlan_offload": "on", 
                  "rx_vlan_stag_filter": "off [fixed]", 
                  "rx_vlan_stag_hw_parse": "off [fixed]", 
                  "scatter_gather": "on", 
                  "tcp_segmentation_offload": "on", 
                  "tx_checksum_fcoe_crc": "off [fixed]", 
                  "tx_checksum_ip_generic": "on", 
                  "tx_checksum_ipv4": "off [fixed]", 
                  "tx_checksum_ipv6": "off [fixed]", 
                  "tx_checksum_sctp": "off [fixed]", 
                  "tx_checksumming": "on", 
                  "tx_fcoe_segmentation": "off [fixed]", 
                  "tx_gre_segmentation": "off [fixed]", 
                  "tx_gso_robust": "off [fixed]", 
                  "tx_ipip_segmentation": "off [fixed]", 
                  "tx_lockless": "off [fixed]", 
                  "tx_mpls_segmentation": "off [fixed]", 
                  "tx_nocache_copy": "off", 
                  "tx_scatter_gather": "on", 
                  "tx_scatter_gather_fraglist": "off [fixed]", 
                  "tx_sctp_segmentation": "off [fixed]", 
                  "tx_sit_segmentation": "off [fixed]", 
                  "tx_tcp6_segmentation": "off [fixed]", 
                  "tx_tcp_ecn_segmentation": "off [fixed]", 
                  "tx_tcp_segmentation": "on", 
                  "tx_udp_tnl_segmentation": "off [fixed]", 
                  "tx_vlan_offload": "on [fixed]", 
                  "tx_vlan_stag_hw_insert": "off [fixed]", 
                  "udp_fragmentation_offload": "off [fixed]", 
                  "vlan_challenged": "off [fixed]"
              }, 
              "hw_timestamp_filters": [], 
              "ipv4": {
                  "address": "192.168.199.142", 
                  "broadcast": "192.168.199.255", 
                  "netmask": "255.255.255.0", 
                  "network": "192.168.199.0"
              }, 
              "ipv6": [
                  {
                      "address": "fe80::3ef1:a048:4d1d:67cd", 
                      "prefix": "64", 
                      "scope": "link"
                  }
              ], 
  	...
  ```

- 假如我们想在一个playbook中针对客户机的某个条件进行判断，例如IP地址，那么就需要使用到上面gather_facts所列出的信息，然后进行判断，上面有一个键名为`ipv4`，内部包含了客户机的IP地址，而ipv4键则包含在`ansible_ens33`键值中，所以使用这个键进行判断，创建`when.yml`playbook，写入内容如下：

  ```yaml
  ---
  - hosts: vm2
    user: root
    gather_facts: True
    tasks:
      - name: use when
        shell: touch /tmp/when.txt
        when: ansible_ens33.ipv4.address == "192.168.199.142"
        # gather_facts中所列的节点使用'.'进行连接，将所有的节点列出来，使用'=='进行判断
  
  ```

- 执行结果如下：

  ```bash
  [root@vm1 ansible]# ansible-playbook when.yml 
  
  PLAY [vm2] ***************************************************************************************
  
  TASK [Gathering Facts] ***************************************************************************
  ok: [vm2]
  
  TASK [use when] **********************************************************************************
   [WARNING]: Consider using file module with state=touch rather than running touch
  
  changed: [vm2]
  
  PLAY RECAP ***************************************************************************************
  vm2                        : ok=2    changed=1    unreachable=0    failed=0   
  
  ```

- 更改判断条件再执行：

  ```yaml
  ---
  - hosts: vm2
    user: root
    gather_facts: True
    tasks:
      - name: use when
        shell: touch /tmp/when.txt
        when: ansible_hostname != "vm2"
  
  ```

  ```bash
  [root@vm1 ansible]# ansible-playbook when.yml 
  
  PLAY [vm2] *************************************************************************************************************************
  
  TASK [Gathering Facts] *************************************************************************************************************
  ok: [vm2]
  
  TASK [use when] ********************************************************************************************************************
  skipping: [vm2]
  
  PLAY RECAP *************************************************************************************************************************
  vm2                        : ok=1    changed=0    unreachable=0    failed=0   
  
  ```

## handlers

- playbook中的handlers相当于shell中的`&&`符号，即前一个命令执行成功后才会执行下一个任务；

- 创建`handlers.yml`，写入以下内容：

  ```yaml
  ---
  - name: handers test
    hosts: vm2
    user: root
    tasks:
      - name: copy file
        copy: src=/etc/passwd dest=/tmp/aaa.txt
        notify: test handlers
    handlers:
      - name: test handlers
        shell: echo "11111" >> /tmp/aaa.txt
  
  ```

- 上面的playbook中，`notify`的值是下面的handlers的`name`的值，handlers则执行shell模块，执行结果如下：

  ```bash
  [root@vm1 ansible]# ansible-playbook handlers.yml 
  
  PLAY [handers test] ******************************************************************************
  
  TASK [Gathering Facts] ***************************************************************************
  ok: [vm2]
  
  TASK [copy file] *********************************************************************************
  changed: [vm2]
  
  RUNNING HANDLER [test handlers] ******************************************************************
  changed: [vm2]
  
  PLAY RECAP ***************************************************************************************
  vm2                        : ok=3    changed=2    unreachable=0    failed=0   
  
  ```

  - vm2的结果：

  ```bash
  [root@vm2 ~]# tail -n 2 /tmp/aaa.txt 
  mysql:x:1103:1103::/home/mysql:/bin/bash
  11111
  
  ```

- handlers会在前面的tasks执行成功之后才会执行，如果前一条命令执行失败，则结果如下：

  ```bash
  [root@vm1 ansible]# ansible-playbook handlers.yml 
  
  PLAY [handers test] ******************************************************************************
  
  TASK [Gathering Facts] ***************************************************************************
  ok: [vm2]
  
  TASK [copy file] *********************************************************************************
  fatal: [vm2]: FAILED! => {"changed": false, "cmd": "pa", "msg": "[Errno 2] 没有那个文件或目录", "rc": 2}
  	to retry, use: --limit @/etc/ansible/handlers.retry
  
  PLAY RECAP ***************************************************************************************
  vm2                        : ok=1    changed=0    unreachable=0    failed=1   
  
  ```

---