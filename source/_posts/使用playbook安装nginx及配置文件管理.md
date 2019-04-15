---
title: 使用playbook安装nginx及配置文件管理
author: Evobot
categories: 自动化运维
tags:
  - ansible
abbrlink: af95245c
date: 2018-10-06 16:40:50
image:
---





本文主要介绍如何使用ansible的playbook功能安装nginx，以及使用ansible管理配置文件。

<!--more-->

---

# 使用playbook安装nginx

## 安装准备

- 首先在一台机器上编译安装好nginx，并且打包好，再使用ansible下发到其他客户机；

- 在ansible服务端的`/etc/ansible`下创建`nginx_install`目录，方便进行管理；

- 进入`nginx_install`目录，执行下面的命令创建需要的目录：

  ```bash
  mkdir -p roles/{common,install}/{handlers,files,meta,tasks,templates,vars}
  ```

- 上面创建的`roles`目录下有两个角色目录，其中`common`目录为一些准备操作，`install`为安装nginx的操作，在两个角色目录下又有几个目录，其中`handlers`下面为当发生改变时需要执行的操作，通常用在配置文件发生改变，重启服务时；`files`为安装时用到的一些文件，`meta`为说明信息，说明角色依赖等信息，可以留空，`tasks`里面是核心的配置文件，`templates`通常存放一些配置文件、启动脚本等模板文件，`vars`下为定义的变量。

  ```bash
  [root@vm1 nginx_install]# tree
  .
  └── roles
      ├── common
      │   ├── files
      │   ├── handlers
      │   ├── meta
      │   ├── tasks
      │   ├── templates
      │   └── vars
      └── install
          ├── files
          ├── handlers
          ├── meta
          ├── tasks
          ├── templates
          └── vars
  
  ```

- 然后进行nginx的编译安装，安装完成后，在nginx中，我们的nginx启动脚本位于`/etc/init.d/nginx`，而nginx的配置文件则位于`/usr/local/nginx/conf/nginx.conf`，然后我们将安装好的nginx进行打包，打包时不需要打包配置文件：

  ```bash
  tar zcvf nginx.tar.gz --exclude "nginx.conf" --exclude "vhosts" nginx/
  ```

- 打包完成后，将压缩包放到`/etc/ansible/nginx_install/roles/install/files`目录下：

  ```bash
  mv nginx.tar.gz /etc/ansible/nginx_install/roles/install/files/
  ```

- 接着将nginx的配置文件`nginx.conf`、启动脚本复制到`roles/install/templates`目录中：

  ```bash
  [root@vm1 local]# cp /usr/local/nginx/conf/nginx.conf /etc/ansible/nginx_install/roles/install/templates/
  [root@vm1 local]# cp /etc/init.d/nginx /etc/ansible/nginx_install/roles/install/templates/
  
  ```

## 编辑playbook

- 进入`/etc/ansible/roles/common`目录，创建`tasks/main.yml`playbook，用来安装nginx依赖的软件包，写入内容如下：

  ```yaml
  - name: Install initializtion require software
    yum: name={{ item }} state=installed
    with_items:
      - zlib-devel
      - pcre-devel
  
  ```

- 接着在`/roles/install/vars`目录下创建`main.yml`playbook，用来定义需要使用到的变量，这些变量可以用来针对不同的客户机，实现分发不同的配置文件，例如配置高的客户机，nginx进程数就设置的多一些，playbook内容如下：

  ```yaml
  nginx_user: www
  nginx_port: 80
  nginx_basedir: /usr/local/nginx
  
  ```

- 然后定义拷贝文件的playbook，创建`roles/install/tasks/copy.yml`文件，写入内容如下：

  ```yaml
  - name: Copy Nginx Software
    copy: src=nginx.tar.gz dest=/tmp/nginx.tar.gz owner=root group=root
  - name: Uncompression Nginx Software
    shell: tar zxf /tmp/nginx.tar.gz -C /usr/local
  - name: Copy Ningx Start Script
    template: src=nginx dest=/etc/init.d/nginx owner=root group=root mode=755
  - name: Copy Nginx Config
    template: src=nginx.conf dest={{ nginx_basedir }}/conf/ owner=root group=root mode=0644
  ```

  - 上面的playbook中，src并没有使用绝对路径，因为ansible的`copy`模块可以到`files`目录下自动寻找文件，而`template`模块则会到`tempates`目录下自动找到文件。

- 创建`roles/install/tasks/install.yml`playbook，用来创建用户，启动服务，删除压缩包，该playbook为总的playbook，内容如下:

  ```yaml
  - name: Create Nginx User
    user: name={{ nginx_user }} state=present createhome=no shell=/sbin/nologin
  - name: Start Nginx Service
    shell: /etc/init.d/nginx start
  - name: Add Boot Start Nignx Service
    shell: chkconfig --level 345 nginx on
  - name: Delete Nginx compression files
    shell: rm -rf /tmp/nginx.tar.gz
  ```

- 然后再在`roles/install/tasks/`目录下创建一个`main.yml`，用来调用copy和install两个脚本，内容如下：

  ```yaml
  - include: copy.yml
  - include: install.yml
  ```

- 上面的步骤完成后，common和install两个roles就已经定义完成，接着需要定义一个入口配置文件，在`/etc/ansible/nginx_install/`目录下创建`install.yml`文件，内容如下：

  ```yaml
  ---
  - hosts: testhost
    remote_user: root
    gather_facts: True
    roles:
      - common
      - install
  ```

  - `roles`模块能够调用指定角色的playbook，利于代码重复调用，例如针对一批主机执行一个角色(common)的任务，针对另一批主机执行另一个角色(install)的任务，或者两个角色都被调用执行。

## 执行playbook

- 执行命令`ansible-playbook /etc/ansible/nginx_install/install.yml`，正确执行的结果如下：

  ```bash
  [root@localhost nginx_install]# ansible-playbook /etc/ansible/nginx_install/install.yml
  [DEPRECATION WARNING]: The use of 'include' for tasks has been deprecated. Use
  'import_tasks' for static inclusions or 'include_tasks' for dynamic inclusions. This
  feature will be removed in a future release. Deprecation warnings can be disabled by
  setting deprecation_warnings=False in ansible.cfg.
  [DEPRECATION WARNING]: include is kept for backwards compatibility but usage is
  discouraged. The module documentation details page may explain more about this
  rationale.. This feature will be removed in a future release. Deprecation warnings can
  be disabled by setting deprecation_warnings=False in ansible.cfg.
  
  PLAY [testhost] *************************************************************************
  
  TASK [Gathering Facts] ******************************************************************
  ok: [centos_2]
  ok: [centos_1]
  
  TASK [common : Install initializtion require software] **********************************
  ok: [centos_1] => (item=[u'zlib-devel', u'pcre-devel', u'gcc'])
  changed: [centos_2] => (item=[u'zlib-devel', u'pcre-devel', u'gcc'])
  
  TASK [install : Copy Nginx Software] ****************************************************
  ok: [centos_1]
  changed: [centos_2]
  
  TASK [install : Uncompression Nginx Software] *******************************************
   [WARNING]: Consider using unarchive module rather than running tar
  
  changed: [centos_1]
  changed: [centos_2]
  
  TASK [install : Copy Nginx Start Script] ************************************************
  ok: [centos_1]
  changed: [centos_2]
  
  TASK [install : Copy Nginx Config] ******************************************************
  ok: [centos_1]
  changed: [centos_2]
  
  TASK [install : Create Nginx User] ******************************************************
  ok: [centos_1]
  changed: [centos_2]
  
  TASK [install : Start Nginx Service] ****************************************************
  changed: [centos_1]
  changed: [centos_2]
  
  TASK [install : Add Boot Start Nginx Service] *******************************************
  changed: [centos_2]
  changed: [centos_1]
  
  TASK [install : Delete Nginx Compression files] *****************************************
   [WARNING]: Consider using file module with state=absent rather than running rm
  
  changed: [centos_2]
  changed: [centos_1]
  
  PLAY RECAP ******************************************************************************
  centos_1                   : ok=10   changed=4    unreachable=0    failed=0
  centos_2                   : ok=10   changed=9    unreachable=0    failed=0
  
  ```

- 如果出现执行时提示yum安装失败的情况，可以将`/etc/ansible/nginx_install/roles/common/tasks/main.yml`文件改成如下形式：

  ```yaml
  - name: Install initializtion require software
    yum: name="pcre-devel,zlib-devel" state=installed
  ```

- 如果在执行到启动nginx时出现失败，注意查看客户机上是否存在80端口被占用的情况。

- 另外需要注意客户机上不能够存在yum安装的nginx软件包，否则启动时会启动yum安装的nginx。



# playbook关系梳理

- 上面安装nginx的playbook，在完成后整个文件及目录关系如下：

  ```bash
  nginx_install/
  ├── install.retry
  ├── install.yml
  └── roles
      ├── common
      │   ├── files
      │   ├── handles
      │   ├── meta
      │   ├── tasks
      │   │   └── main.yml
      │   ├── templates
      │   └── vars
      └── install
          ├── files
          │   └── nginx.tar.gz
          ├── handles
          ├── meta
          ├── tasks
          │   ├── copy.yml
          │   ├── install.yml
          │   └── main.yml
          ├── templates
          │   ├── nginx
          │   └── nginx.conf
          └── vars
              └── main.yml
  
  ```

- 首先`nginx_install`目录下的总入口文件`install.yml`定义了执行的主机组以及需要执行的角色`roles`，根据roles的定义，首先执行了`common`下的任务；

- `common`角色会在`tasks`目录下找到`main.yml`的playbook进行执行，完成后再执行`install`角色的playbook；

- `install`角色的playbook中，同样会再`tasks`目录下先执行`main.yml`，根据该playbook中定义的过程，会先执行`copy.yml`，然后执行`install.yml`；

- 而`install`目录下的`files`、`templates`用来存放playbook需要用到的文件，`vars`则存放变量的playbook。

- 整体流程图如下：

![nginx_playbook](https://blogimage-1251925320.cos.ap-chengdu.myqcloud.com/nginx_playbook.png)

---

# playbook管理配置文件

## 管理配置文件更新

- 生产环境中大多数时候是需要管理配置文件的，安装软件包只是在初始化环境的时候使用，而配置文件则是在使用过程中非常常用的，例如配置的下发、回滚等等；
- 我们可以使用playbook来实现对配置文件的管理，例如管理nginx配置的playbook，具体步骤如下：

1. 在`/etc/ansible/`目录下创建以下目录：

   ```bash
   mkdir -p /etc/ansible/nginx_config/roles/{new,old}/{files,handlers,vars,tasks}
   ```

   其中new目录为更新时用到的，old则是回滚时用到的，files下面为nginx.conf和vhosts目录，handlers为重启nginx服务的命令

2. 在执行playbook前一定要先备份一下旧的配置，对于老的配置文件的管理一定要严格，不要随意修改线上机器的配置，并且要保证`new/files`下面的配置和线上机器的配置一致；

3. 首先将`nginx.conf`和`vhost`目录拷贝到`/etc/ansible/nginx_config/roles/new/files/`目录下：

   ```bash
   cd /usr/local/nginx/conf
   cp -r nginx.conf vhosts/ /etc/ansible/nginx_config/roles/new/files/
   ```

4. 然后在`roles/new/vars/`目录下创建定义变量的`main.yml`文件，写入如下内容：

   ```yaml
   nginx_basedir:/usr/local/nginx
   ```

5. 接着在`roles/new/handlers/`目录下创建定义重新加载nginx服务的`main.yml`文件，写入内容如下：

   ```yaml
   - name:restart nginx
     shell:/etc/init.d/nginx reload
   ```

6. 然后在`roles/new/tasks/`目录下创建核心人物的`main.yml`文件，内容如下：

   ```yaml
   - name: copy conf file
     copy: src={{ item.src }} dest={{ nginx_basedir }}/{{ item.dest }} backup=yes owner=root group=root m
   ode=0644
     with_items:
       - { src: nginx.conf, dest: conf/nginx.conf }
       - { src: vhosts, dest: conf/ }
     notify: restart nginx
   
   ```

   这里的`{{ item.src }}`和`{{ item.dest }}`使用了两个循环变量，并且在下面的`with_items`中分别定义了两次循环的src和dest变量，最后再执行`restart nginx`这个handlers。

7. 最后在`/etc/ansible/nginx_config/`目录下定义总入口配置文件`update.yml`，其内容如下：

   ```yaml
   ---
   - hosts: testhost
     user: root
     roles:
       - new
   ```

8. 执行playbook，查看是否成功：

   ```bash
   [root@localhost nginx_config]# ansible-playbook /etc/ansible/nginx_config/update.yml
   
   PLAY [testhost] **************************************************************************************
   
   TASK [Gathering Facts] *******************************************************************************
   ok: [centos_2]
   ok: [centos_1]
   
   TASK [new : copy conf file] **************************************************************************
   changed: [centos_2] => (item={u'dest': u'conf/nginx.conf', u'src': u'nginx.conf'})
   changed: [centos_1] => (item={u'dest': u'conf/nginx.conf', u'src': u'nginx.conf'})
   changed: [centos_1] => (item={u'dest': u'conf/', u'src': u'vhosts'})
   changed: [centos_2] => (item={u'dest': u'conf/', u'src': u'vhosts'})
   
   RUNNING HANDLER [new : restart nginx] ****************************************************************
   changed: [centos_2]
   changed: [centos_1]
   
   PLAY RECAP *******************************************************************************************
   centos_1                   : ok=3    changed=2    unreachable=0    failed=0
   centos_2                   : ok=3    changed=2    unreachable=0    failed=0
   
   ```

9. 我们也可以对`/role/new/files/nginx.conf`做一些修改，然后重新执行playbook，再查看执行结果：

   ```bash
   [root@localhost new]# ansible-playbook /etc/ansible/nginx_config/update.yml
   
   PLAY [testhost] **************************************************************************************
   
   TASK [Gathering Facts] *******************************************************************************
   ok: [centos_2]
   ok: [centos_1]
   
   TASK [new : copy conf file] **************************************************************************
   changed: [centos_1] => (item={u'dest': u'conf/nginx.conf', u'src': u'nginx.conf'})
   changed: [centos_2] => (item={u'dest': u'conf/nginx.conf', u'src': u'nginx.conf'})
   ok: [centos_1] => (item={u'dest': u'conf/', u'src': u'vhosts'})
   ok: [centos_2] => (item={u'dest': u'conf/', u'src': u'vhosts'})
   
   RUNNING HANDLER [new : restart nginx] ****************************************************************
   changed: [centos_1]
   changed: [centos_2]
   
   PLAY RECAP *******************************************************************************************
   centos_1                   : ok=3    changed=2    unreachable=0    failed=0
   centos_2                   : ok=3    changed=2    unreachable=0    failed=0
   
   
   ```

   可以看到，只更改nginx.conf时，playbook对远程机器也会只更改nginx.conf。

## 管理配置文件回滚

- 回滚操作就是用旧的配置覆盖现有的配置，然后重新加载nginx服务，每次改动nginx配置文件之前，应该先将配置备份到`old/files`目录中；

- 首先备份旧的配置文件，使用`rsync`命令将配置同步到`old/files`下：

  ```bash
  rsync -av /etc/ansible/nginx_config/roles/new/ /etc/ansible/nginx_config/roles/old/
  ```

- 然后在`nginx_config`目录下创建`rollback.yml`总入口文件，内容如下：

  ```yaml
  ---
  - hosts: testhost
    user: root
    roles:
      - old
  
  ```

- 最后执行`ansible-playbook rollback.yml`即可完成配置的回滚，实际上回滚与更新只改变了配置文件，其余的操作与更新是相同的，所以只需要在playbook的总入口文件中指定使用old角色即可。

---

