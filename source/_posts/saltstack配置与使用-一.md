---
title: saltstack配置与使用(一)
author: Evobot
date: 2018-09-19 21:04:56
categories: 自动化运维
tags:
  - saltstack
image:
---

1. saltstack远程执行命令
2. grains
3. pillar
4. 使用saltstack安装配置httpd
5. 配置管理文件

<!--more-->

---

# saltstack远程执行命令

##salt命令及常用选项

- 在服务端可以使用`salt ‘*’ test.ping`命令来检测客户端是否存活，`‘*’`表示所有的客户端，如果只想检测制定的客户端，也可以使用客户端的hostname，如`salt ‘vm2’ test.ping`：

  ```bash
  [root@vm1 ~]# salt '*' test.ping
  vm2:
      True
  vm1:
      True
  
  ```

- saltstack想要在客户机执行系统命令，则使用`salt ‘*’ cmd.run “command”`的形式进行执行，例如查看客户端的hostname：

  ```bash
  salt '*' cmd.run "hostname"
  ```

  ```bash
  [root@vm1 ~]# salt '*' cmd.run "hostname"
  vm2:
      vm2
  vm1:
      vm1
  [root@vm1 ~]# salt vm2 cmd.run "ip ad"
  vm2:
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
          inet6 ::1/128 scope host 
             valid_lft forever preferred_lft forever
      2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
          link/ether 00:0c:29:cd:62:0c brd ff:ff:ff:ff:ff:ff
          inet 192.168.49.129/24 brd 192.168.49.255 scope global ens32
             valid_lft forever preferred_lft forever
          inet6 fe80::e2ac:d368:af2c:1e12/64 scope link 
             valid_lft forever preferred_lft forever
  
  ```

- 上面执行的salt命令，除了可以使用`*`匹配所有已认证的主机，可以使用`salt-key`查看所有认证的主机，还可以使用正则匹配、列表、通配的形式：

  - 例如对于vm1和vm2主机，使用`salt ‘vm*’ cmd.run “cmd”`匹配以vm开头的主机；
  - `salt ‘vm[12]’ cmd.run “cmd”`则匹配vm后面是1或者是2 的主机；
  - `salt -L ‘vm1,vm2’ cmd.run “cmd”`，其中`-L`表示列表，在选项后面的主机名使用逗号分隔；
  - `salt -E ‘vm[0-9]+’ cmd.run “cmd”`，其中`-E`表示正则匹配，后面写主机名的正则匹配规则；

  ```bash
  [root@vm1 ~]# salt 'vm[12]' test.ping
  vm2:
      True
  vm1:
      True
      
  [root@vm1 ~]# salt 'vm*' cmd.run 'pwd'
  vm2:
      /root
  vm1:
      /root
      
  [root@vm1 ~]# salt -L 'vm1,vm2' cmd.run 'w'
  vm1:
       21:31:35 up 32 min,  1 user,  load average: 0.07, 0.06, 0.11
      USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
      root     pts/0    192.168.49.1     21:07    7.00s  0.70s  0.60s /usr/bin/python /usr/bin/salt -L vm1,vm2 cmd.run w
  vm2:
       21:31:35 up 32 min,  1 user,  load average: 0.14, 0.07, 0.06
      USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
      root     pts/0    192.168.49.1     21:07   23:03   0.02s  0.02s -bash
  
  [root@vm1 ~]# salt -E 'vm[0-9]+' test.ping
  vm2:
      True
  vm1:
      True
  
  ```

- 另外salt命令还支持`-G`grains选项和`-l`pillar选项，下面会对这两个选项进行介绍。

## grains

### grains配置

- grains是在minion启动时收集到的一些信息，比如操作系统类型、网卡IP、内核版本，CPU架构等等；

- 使用`salt ‘hostname’ grains.ls`命令可以查看指定主机的grains信息列表：

  ```bash
  [root@vm1 ~]# salt 'vm2' grains.ls
  vm2:
      - SSDs
      - biosreleasedate
      - biosversion
      - cpu_flags
      - cpu_model
      - cpuarch
      - disks
      - dns
      - domain
      - fqdn
      - fqdn_ip4
      - fqdn_ip6
      - gid
      - gpus
      - groupname
      - host
      - hwaddr_interfaces
      - id
      - init
      - ip4_gw
      - ip4_interfaces
      - ip6_gw
      - ip6_interfaces
      - ip_gw
      - ip_interfaces
      - ipv4
      - ipv6
      - kernel
      - kernelrelease
      - kernelversion
      - locale_info
      - localhost
      - lsb_distrib_codename
      - lsb_distrib_id
      - machine_id
      - manufacturer
      - master
      - mdadm
      - mem_total
      - nodename
      - num_cpus
      - num_gpus
      - os
      - os_family
      - osarch
      - oscodename
      - osfinger
      - osfullname
      - osmajorrelease
      - osrelease
      - osrelease_info
      - path
      - pid
      - productname
      - ps
      - pythonexecutable
      - pythonpath
      - pythonversion
      - saltpath
      - saltversion
      - saltversioninfo
      - selinux
      - serialnumber
      - server_id
      - shell
      - swap_total
      - systemd
      - uid
      - username
      - uuid
      - virtual
      - zfs_feature_flags
      - zfs_support
      - zmqversion
  
  ```

- 如果要查看grains收集到的信息的值，则使用`salt 'hostname' grains.items`:

  ```bash
  [root@vm1 ~]# salt 'vm2' grains.items
  vm2:
      ----------
      SSDs:
      biosreleasedate:
          05/19/2017
      biosversion:
          6.00
      cpu_flags:
          - fpu
          - vme
          - de
          - pse
          - tsc
          - msr
          - pae
          - mce
          - cx8
          - apic
          - sep
          - mtrr
          - pge
          - mca
          - cmov
          - pat
          - pse36
          - clflush
          - mmx
          - fxsr
          - sse
          - sse2
          - ss
          - syscall
          - nx
          - rdtscp
          - lm
          - constant_tsc
          - arch_perfmon
          - nopl
          - xtopology
          - tsc_reliable
          - nonstop_tsc
          - pni
          - pclmulqdq
          - ssse3
          - cx16
          - pcid
          - sse4_1
          - sse4_2
          - x2apic
          - popcnt
          - tsc_deadline_timer
          - aes
          - xsave
          - avx
          - f16c
          - rdrand
          - hypervisor
          - lahf_lm
          - arat
          - fsgsbase
          - tsc_adjust
          - smep
      cpu_model:
          Intel(R) Core(TM) i5-3230M CPU @ 2.60GHz
      cpuarch:
          x86_64
      disks:
          - sda
          - sr0
      dns:
          ----------
          domain:
          ip4_nameservers:
              - 114.114.114.114
          ip6_nameservers:
          nameservers:
              - 114.114.114.114
          options:
          search:
          sortlist:
      domain:
      fqdn:
          vm2
      fqdn_ip4:
          - 192.168.49.129
      fqdn_ip6:
          - fe80::e2ac:d368:af2c:1e12
      gid:
          0
      gpus:
          |_
            ----------
            model:
                SVGA II Adapter
            vendor:
                unknown
      groupname:
          root
      host:
          vm2
      hwaddr_interfaces:
          ----------
          ens32:
              00:0c:29:cd:62:0c
          lo:
              00:00:00:00:00:00
      id:
          vm2
      init:
          systemd
      ip4_gw:
          192.168.49.2
      ip4_interfaces:
          ----------
          ens32:
              - 192.168.49.129
          lo:
              - 127.0.0.1
      ip6_gw:
          False
      ip6_interfaces:
          ----------
          ens32:
              - fe80::e2ac:d368:af2c:1e12
          lo:
              - ::1
      ip_gw:
          True
      ip_interfaces:
          ----------
          ens32:
              - 192.168.49.129
              - fe80::e2ac:d368:af2c:1e12
          lo:
              - 127.0.0.1
              - ::1
      ipv4:
          - 127.0.0.1
          - 192.168.49.129
      ipv6:
          - ::1
          - fe80::e2ac:d368:af2c:1e12
      kernel:
          Linux
      kernelrelease:
          3.10.0-514.el7.x86_64
      kernelversion:
          #1 SMP Tue Nov 22 16:42:41 UTC 2016
      locale_info:
          ----------
          defaultencoding:
              UTF-8
          defaultlanguage:
              zh_CN
          detectedencoding:
              UTF-8
      localhost:
          vm2
      lsb_distrib_codename:
          CentOS Linux 7 (Core)
      lsb_distrib_id:
          CentOS Linux
      machine_id:
          a482f7e958b347d68d4f3a097276cafc
      manufacturer:
          VMware, Inc.
      master:
          vm1
      mdadm:
      mem_total:
          472
      nodename:
          vm2
      num_cpus:
          1
      num_gpus:
          1
      os:
          CentOS
      os_family:
          RedHat
      osarch:
          x86_64
      oscodename:
          CentOS Linux 7 (Core)
      osfinger:
          CentOS Linux-7
      osfullname:
          CentOS Linux
      osmajorrelease:
          7
      osrelease:
          7.3.1611
      osrelease_info:
          - 7
          - 3
          - 1611
      path:
          /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
      pid:
          2161
      productname:
          VMware Virtual Platform
      ps:
          ps -efHww
      pythonexecutable:
          /usr/bin/python
      pythonpath:
          - /usr/bin
          - /usr/lib64/python27.zip
          - /usr/lib64/python2.7
          - /usr/lib64/python2.7/plat-linux2
          - /usr/lib64/python2.7/lib-tk
          - /usr/lib64/python2.7/lib-old
          - /usr/lib64/python2.7/lib-dynload
          - /usr/lib64/python2.7/site-packages
          - /usr/lib/python2.7/site-packages
      pythonversion:
          - 2
          - 7
          - 5
          - final
          - 0
      saltpath:
          /usr/lib/python2.7/site-packages/salt
      saltversion:
          2018.3.2
      saltversioninfo:
          - 2018
          - 3
          - 2
          - 0
      selinux:
          ----------
          enabled:
              True
          enforced:
              Permissive
      serialnumber:
          VMware-56 4d a9 64 41 53 26 45-b4 01 d3 f3 a4 cd 62 0c
      server_id:
          147626588
      shell:
          /bin/sh
      swap_total:
          2047
      systemd:
          ----------
          features:
              +PAM +AUDIT +SELINUX +IMA -APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTSETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 -SECCOMP +BLKID +ELFUTILS +KMOD +IDN
          version:
              219
      uid:
          0
      username:
          root
      uuid:
          64a94d56-5341-4526-b401-d3f3a4cd620c
      virtual:
          VMware
      zfs_feature_flags:
          False
      zfs_support:
          False
      zmqversion:
          4.1.4
  
  ```

- 这些grains信息，可以帮助我们进行资产信息管理；grains信息不是动态变更的，只会在minion启动时进行收集；

- grains支持自定义信息，需要在minion上创建并编辑配置文件`/etc/salt/grains`，写入以下配置：

  ```bash
  //key:value形式自定义
  role: nginx
  env: test
  
  ```

  - 配置完成后，重启minion服务。

###grains使用

- 重启客户机minion服务之后，在服务端master上可以执行`salt '*' grains.item role env`命令查看自定义的grains项：

  ```bash
  [root@vm1 ~]# salt '*' grains.item role env
  vm2:
      ----------
      env:
          test
      role:
          nginx
  vm1:
      ----------
      env:
      role:
  
  ```

- 通过对grains的了解，我们在远程执行命令时，可以使用`salt -G key:value cmd.run “cmd”`对主机进行匹配来执行命令：

  ```bash
  [root@vm1 ~]# salt -G role:nginx cmd.run "ls"
  vm2:
      myproject
      sample
  
  [root@vm1 pillar]# salt -G ip4_interfaces:ens32:192.168.49.129 cmd.run "ip addr"
  vm2:
      1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
          link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
          inet 127.0.0.1/8 scope host lo
             valid_lft forever preferred_lft forever
          inet6 ::1/128 scope host 
             valid_lft forever preferred_lft forever
      2: ens32: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
          link/ether 00:0c:29:cd:62:0c brd ff:ff:ff:ff:ff:ff
          inet 192.168.49.129/24 brd 192.168.49.255 scope global ens32
             valid_lft forever preferred_lft forever
          inet6 fe80::e2ac:d368:af2c:1e12/64 scope link 
             valid_lft forever preferred_lft forever
  
  ```

- 通过这样的方式，可以对一些机器使用grains自定义的方式进行分组，从而针对分组批量远程执行命令。

## pillar

###pillar配置

- pillar和grains类似，但pillar是在master上定义的，并且是针对minion定义的一些信息，例如比较重要的数据（密码）可以存在pillar里，也可以自定义变量等；

- 配置自定义pillar，需要修改master的配置文件`/etc/salt/pillar`，找到`pillar_roots`配置项，去掉配置项及配置的注释，配置如下：

  ```bash
  pillar_roots:
    base:
      - /srv/pillar
  
  ```

- 保存配置，然后重启`salt-master`服务，再创建上面配置中指定目录`/srv/pillar`：

  ```bash
  mkdir -p /src/pillar/
  ```

- 切换到`/srv/pillar`目录，然后创建一个saltstack配置文件，以`.sls`后缀名结尾：

  ```bash
  vi test.sls
  ```

  写入以下配置内容定义子配置文件，格式同样是key:value形式：

  ```bash
  conf: /etc/123.conf
  ```

- 再创建一个`top.sls`配置文件，作为saltstack的pillar主入口文件：

  ```bash
  vi top.sls
  ```

  写入配置如下：

  ```bash
  base:
    'vm2':	#指定主机名
      - test	#指定加载的配置文件名，多个配置文件以同样格式写多行
      - test2
  ```

  也可以为多个主机指定需要加载的配置文件，格式如下：

  ```bash
  base:
    'vm2':	
      - test
    'vm1'
      - test2
  ```

  ```bash
  //test2.sls内容,配置也可以写到一个文件中
  dir: /usr/local
  ```

  > pillar的配置文件需要特别注意格式，空格不能少。

- 接着为了使配置的pillar生效，需要执行`salt ‘*’ saltutil.refresh_pillar`命令刷新pillar：

  ```bash
  [root@vm1 pillar]# salt '*' saltutil.refresh_pillar
  vm2:
      True
  vm1:
      True
  
  ```

###pillar使用

- 刷新pillar之后，我们就可以使用命令`salt ‘*’ pillar.item [key]`来查看指定的pillar：

  ```bash
  [root@vm1 pillar]# salt '*' pillar.item dir
  vm2:
      ----------
      dir:
          /usr/local
  vm1:
      ----------
      dir:
  [root@vm1 pillar]# salt '*' pillar.item conf
  vm2:
      ----------
      conf:
          /etc/123.conf
  vm1:
      ----------
      conf:
  
  ```

- 使用`salt -I ‘key:value’ cmd.run “cmd”`命令可以用pillar对主机进行匹配并执行命令：

  ```bash
  [root@vm1 pillar]# salt -I 'conf:/etc/123.conf' cmd.run "ls"
  vm2:
      myproject
      sample
  
  ```

# 使用saltstack安装配置httpd

##配置自动化安装httpd

- 我们使用saltstack来远程自动化为客户端安装httpd软件包并进行配置；

- 首先在master上打开`/etc/salt/master`配置文件，找到`file_roots`配置项，打开配置项及其配置的注释，如下：

  ```bash
  file_roots:
    base:
      - /srv/salt
  
  ```

- 然后创建上面的配置指定的目录`/srv/salt/`，进入目录，创建`top.sls`总入口配置文件：

  ```bash
  mkdir -p /srv/salt/
  cd !$
  vi top.sls
  ```

- 在top.sls中写入以下配置，格式与上面的pillar配置文件相同，指定主机名，然后指定配置文件名：

  ```bash
  base:
    '*':
      - httpd
  ```

- 保存top.sls配置文件后重启salt-master服务：

  ```bash
  systemctl restart salt-master
  ```

- 接着在`/srv/salt/`目录下继续创建`httpd.sls`配置文件，写入以下内容：

  ```bash
  httpd-service:	#表示定义的服务或配置的名字
    pkg.installed:	#saltstack内置模块，类似于命令行的cmd.run，用来安装软件包，对不同的操作系统，saltstack会自动调用系统安装软件的命令，如redhat系的yum和debian系的apt-get
      - names:	#指定要安装的软件包的名字
        - httpd
        - httpd-devel
    service.running:	#saltstack内置模块，用来启动指定的服务
      - name: httpd	#需要启动的服务名，centos6的服务在/etc/init.d/目录下，centos7在/lib/systemd/system/目录下
      - enable: True	#是否启动服务，True表示启动，False表示关闭
  
  ```

## 执行自动化安装httpd

- 使用`salt ‘hostname’ state.highstate`命令，saltstack会自动读取`/srv/salt/top.sls`文件，并根据配置，到指定主机完成自动化安装过程：

  ```bash
  salt 'vm2' state.highstate
  # 命令执行过程可能会比较长，需要等待
  ```

- 在被执行的主机上，可以看到对应的yum进程已经开始执行：

  ```bash
  [root@vm2 ~]# ps aux |grep yum
  root       4014  0.0  5.0 321416 24496 ?        S    00:07   0:00 /usr/bin/python /usr/bin/yum --quiet --assumeyes check-update --setopt=autocheck_running_kernel=false
  
  ```

- 安装完成后。master上显示如下：

  ```bash
  [root@vm1 salt]# salt 'vm2' state.highstate
  vm2:
  ----------
            ID: httpd-service
      Function: pkg.installed
          Name: httpd
        Result: True
       Comment: All specified packages are already installed
       Started: 00:07:31.797852
      Duration: 5310.557 ms
       Changes:   
  ----------
            ID: httpd-service
      Function: pkg.installed
          Name: httpd-devel
        Result: True
       Comment: The following packages were installed/updated: httpd-devel
       Started: 00:07:37.108731
      Duration: 38266.074 ms
       Changes:   
                ----------
                apr-devel:
                    ----------
                    new:
                        1.4.8-3.el7_4.1
                    old:
                apr-util-devel:
                    ----------
                    new:
                        1.5.2-6.el7
                    old:
                cyrus-sasl:
                    ----------
                    new:
                        2.1.26-23.el7
                    old:
                cyrus-sasl-devel:
                    ----------
                    new:
                        2.1.26-23.el7
                    old:
                cyrus-sasl-lib:
                    ----------
                    new:
                        2.1.26-23.el7
                    old:
                        2.1.26-20.el7_2
                expat:
                    ----------
                    new:
                        2.1.0-10.el7_3
                    old:
                        2.1.0-8.el7
                expat-devel:
                    ----------
                    new:
                        2.1.0-10.el7_3
                    old:
                httpd-devel:
                    ----------
                    new:
                        2.4.6-80.el7.centos.1
                    old:
                libdb:
                    ----------
                    new:
                        5.3.21-24.el7
                    old:
                        5.3.21-19.el7
                libdb-devel:
                    ----------
                    new:
                        5.3.21-24.el7
                    old:
                libdb-utils:
                    ----------
                    new:
                        5.3.21-24.el7
                    old:
                        5.3.21-19.el7
                openldap:
                    ----------
                    new:
                        2.4.44-15.el7_5
                    old:
                        2.4.40-13.el7
                openldap-devel:
                    ----------
                    new:
                        2.4.44-15.el7_5
                    old:
  ----------
            ID: httpd-service
      Function: service.running
          Name: httpd
        Result: True
       Comment: Service httpd has been enabled, and is running
       Started: 00:08:18.307970
      Duration: 1049.841 ms
       Changes:   
                ----------
                httpd:
                    True
  
  Summary for vm2
  ------------
  Succeeded: 3 (changed=2)
  Failed:    0
  ------------
  Total states run:     3
  Total run time:  44.626 s
  
  ```

- 在客户端主机上，可以看到httpd服务已经运行：

  ```bash
  [root@vm2 ~]# ps aux|grep httpd
  root       4176  0.0  1.0 224032  5028 ?        Ss   00:08   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     4179  0.0  0.6 224032  2952 ?        S    00:08   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     4180  0.0  0.6 224032  2952 ?        S    00:08   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     4181  0.0  0.6 224032  2952 ?        S    00:08   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     4182  0.0  0.6 224032  2952 ?        S    00:08   0:00 /usr/sbin/httpd -DFOREGROUND
  apache     4183  0.0  0.6 224032  2952 ?        S    00:08   0:00 /usr/sbin/httpd -DFOREGROUND
  
  [root@vm2 ~]# ls /lib/systemd/system/httpd.service 
  /lib/systemd/system/httpd.service
  
  ```

# 文件分发

## 介绍及配置

- saltstack不仅可以自动化安装服务，还可以对配置文件进行管理和分发，例如需要对minion的某个服务配置进行修改，可以在master上进行对应的配置，然后将配置文件自动分发到指定的主机，并且进行一些后续的服务操作，如重启服务等；

- 首先在master上创建`/srv/salt/test.sls`配置文件，写入以下内容：

  ```bash
  file_test:	#任务名称
    file.managed:	# saltstack模块
      - name: /tmp/evobot.conf	#minion端的文件
      - source: salt://test/123/1.txt	#指定来源文件.salt://表示当前配置文件的根目录，也就是/etc/salt/master配置文件中定义的/srv/salt/目录
      - user: root	#定义文件属主
      - group: root	#定义文件数组
      - mode: 600	#定义文件权限
  ```

- 创建配置中指定的源配置文件目录，并将要分发的配置文件放入该目录下：

  ````bash
  mkdir -p /srv/salt/test/123
  cp /etc/inittab /srv/salt/test/123/1.txt
  ````

- 然后修改top.sls配置文件，指定加载test.sls配置文件：

  ```bash
  base:
    '*':
      - test
  ```

## 执行配置分发

- 使用命令`salt ‘hostname’ state.highstate`执行自动化分发：

  ```bash
  [root@vm1 123]# salt 'vm2' state.highstate
  vm2:
  ----------
            ID: file_test
      Function: file.managed
          Name: /tmp/evobot.conf
        Result: True
       Comment: File /tmp/evobot.conf updated
       Started: 00:28:55.900935
      Duration: 226.69 ms
       Changes:   
                ----------
                diff:
                    New file
  
  Summary for vm2
  ------------
  Succeeded: 1 (changed=1)
  Failed:    0
  ------------
  Total states run:     1
  Total run time: 226.690 ms
  
  ```

- 到客户机查看是否已经有了evobot.conf文件：

  ```bash
  [root@vm2 ~]# ls -l /tmp/evobot.conf 
  -rw-------. 1 root root 511 9月  20 00:28 /tmp/evobot.conf
  [root@vm2 ~]# cat /tmp/evobot.conf 
  # inittab is no longer used when using systemd.
  #
  # ADDING CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
  #
  # Ctrl-Alt-Delete is handled by /usr/lib/systemd/system/ctrl-alt-del.target
  #
  # systemd uses 'targets' instead of runlevels. By default, there are two main targets:
  #
  # multi-user.target: analogous to runlevel 3
  # graphical.target: analogous to runlevel 5
  #
  # To view current default target, run:
  # systemctl get-default
  #
  # To set a default target, run:
  # systemctl set-default TARGET.target
  #
  
  ```



  