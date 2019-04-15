---
title: Docker模板创建镜像及容器、仓库和数据管理
author: Evobot
categories: 虚拟化与容器化
tags:
  - Docker
abbrlink: 1af6f2ec
date: 2019-04-08 21:56:43
image:
---

1. 通过模板创建镜像
2. 容器管理
3. 仓库管理
4. 数据管理

<!--more-->

---

# 通过模板创建镜像

## 创建镜像

- 首先下载一个模板，可以到[openvz站点](https://wiki.openvz.org/Download/template/precreated)下载，这里下载[centos6的最小镜像模板](http://download.openvz.org/template/precreated/centos-6-x86_64-minimal.tar.gz)。

- 然后使用`cat [模板文件名] | docker import - [镜像名]`命令导入镜像，如下：

  ```bash
  [root@localhost ~]# cat centos-6-x86_64-minimal.tar.gz | docker import - centos6
  sha256:2f7d7ab8be8c6e6cc4cfc23d3c5b39584227db193c0b774dfd29a60a181987bf
  
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos6             latest              2f7d7ab8be8c        39 seconds ago      553MB
  centos_with_net     latest              52992c2b5487        3 days ago          285MB
  centos              latest              9f38484d220f        3 weeks ago         202MB
  
  ```

- 然后可以运行这个镜像，查看版本是否为centos6：

  ```bash
  [root@localhost ~]# docker run -itd centos6 bash
  5ff5032001fe2d79445dd006e6ffbbd05be87355fc79da861d68cbb94ad09be3
  [root@localhost ~]# docker exec -it 5ff503 bash
  [root@5ff5032001fe /]# cat /etc/issue
  CentOS release 6.8 (Final)
  Kernel \r on an \m
  
  
  ```

  

## 导出及恢复镜像

- docker中，不仅可以通过模板导入镜像，也可以将镜像导出为文件；

- 导出镜像使用`docker save -o [导出文件名].tar [镜像名]`命令，如下：

  ```bash
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos6             latest              2f7d7ab8be8c        10 minutes ago      553MB
  centos_with_net     latest              52992c2b5487        3 days ago          285MB
  centos              latest              9f38484d220f        3 weeks ago         202MB
  
  [root@localhost ~]# docker save -o centos7_with_nettool.tar centos_with_net
  [root@localhost ~]# ls
  anaconda-ks.cfg                 centos7_with_nettool.tar
  
  ```

- 导出的镜像文件，可以用来恢复镜像，使用`docker load --input [镜像文件名].tar` 或者`docker load < [镜像文件].tar`，如下：

  ```bash
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos6             latest              2f7d7ab8be8c        18 minutes ago      553MB
  centos_with_net     latest              52992c2b5487        3 days ago          285MB
  centos              latest              9f38484d220f        3 weeks ago         202MB
  
  #删除已存在的镜像，报错镜像被使用无法删除
  [root@localhost ~]# docker rmi 52992c2b5487
  Error response from daemon: conflict: unable to delete 52992c2b5487 (cannot be forced) - image is being used by running container fc0e7fef4089
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS        PORTS               NAMES
  5ff5032001fe        centos6             "bash"              14 minutes ago      Up 14 minutes                           affectionate_ramanujan
  fc0e7fef4089        centos_with_net     "bash"              3 days ago          Up 3 days                            wonderful_gates
  b27af36dca0b        centos              "/bin/bash"         3 days ago          Up 3 days                            reverent_joliot
  
  # -f强制删除使用镜像的容器
  [root@localhost ~]# docker rm -f fc0e7fef4089
  fc0e7fef4089
  
  #删除镜像
  [root@localhost ~]# docker rmi 52992c2b5487
  Untagged: centos_with_net:latest
  Deleted: sha256:52992c2b54875c47ffe7347424016b88286d0c5b5a7ce316662eb9f31f1217e2
  Deleted: sha256:734dfd9c2e9642d133f5b6c9b10d3a0424e25c8b9b24b2f282e5ed05f529a7df
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos6             latest              2f7d7ab8be8c        19 minutes ago      553MB
  centos              latest              9f38484d220f        3 weeks ago         202MB
  
  #从镜像文件恢复镜像
  [root@localhost ~]# docker load --input centos7_with_nettool.tar
  be5de7957e8c: Loading layer  83.78MB/83.78MB
  Loaded image: centos_with_net:latest
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos6             latest              2f7d7ab8be8c        19 minutes ago      553MB
  centos_with_net     latest              52992c2b5487        3 days ago          285MB
  centos              latest              9f38484d220f        3 weeks ago         202MB
  
  ```

- docker可以将自己的镜像推送到官方仓库，使用`docker push [image_name] `命令，但推送之前需要在docker仓库注册。

# 容器管理

- `docker create -it [image_name] bash`命令可以创建一个容器，但创建的容器并未运行，类似于创建了一个没有开机的虚拟机，创建的容器使用`docker ps`是查看不到的，需要使用`docker ps -a`命令来查看：

  ```bash
  [root@localhost ~]# docker create -it centos6 bash
  8a758618ab88d242a920532017cf19bf1981299266405de05767905254d80e53
  
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  b27af36dca0b        centos              "/bin/bash"         3 days ago          Up 3 days                               reverent_joliot
  
  [root@localhost ~]# docker ps -a
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  8a758618ab88        centos6             "bash"              37 seconds ago      Created                                 thirsty_sinoussi
  b27af36dca0b        centos              "/bin/bash"         3 days ago          Up 3 days                               reverent_joliot
  
  ```

  可以看到创建的容器centos6的状态是`Created`。

- 启动创建的容器，使用`docker start [CONTAINER ID]`，如下：

  ```bash
  [root@localhost ~]# docker start 8a758618ab88
  8a758618ab88
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  8a758618ab88        centos6             "bash"              2 minutes ago       Up 2 seconds                            thirsty_sinoussi
  
  ```

  - 相应的，容器有`start`操作，也有`stop`、`restart`操作；

- `docker run -it [image_name] bash`命令在不使用`-d`后台运行选项的情况下，会直接进入容器的虚拟终端，在终端中可以运行命令，但使用`exit`或者`ctrl+d`退出容器后，容器也会立即停止：

  ```bash
  [root@localhost ~]# docker run -it centos bash
  [root@04919c9dea63 /]# ip addr
  bash: ip: command not found
  [root@04919c9dea63 /]# exit
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  [root@localhost ~]# docker ps -a
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                       PORTS               NAMES
  04919c9dea63        centos              "bash"              29 seconds ago      Exited (127) 5 seconds ago                       hopeful_blackwell
  
  ```

  可以看到，退出容器后，使用`docker ps`查看不到容器，`docker ps -a`查看到的容器状态是`Exited`

- 运行容器时，使用的`-i`选项表示以交互模式运行容器，`-t`表示为容器分配一个伪终端，这两个选项通常同时使用，如果不使用这两个选项，单独使用`-d`选项，则命令可以为`docker run -d centos bash -c "while :;do echo "123";sleep 2;done`这种形式，这种形式会直接在后台运行指定的命令，一般很少使用这种形式；

- `docker run --name [container_name] -itd [image_name] bash`命令中，`--name`选项可以为容器指定一个名字，默认不使用该选项时，docker会使用随机字符串为容器命名，为容器命名后，可以直接使用容器名进入容器：

  ```bash
  [root@localhost ~]# docker run -itd --name centos6_1 centos6 bash
  15f4870107f0a1b32b4e3448d949fbf3d1915b90ba64cbf54dec311872231c0e
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  15f4870107f0        centos6             "bash"              3 seconds ago       Up 2 seconds                            centos6_1
  
  [root@localhost ~]# docker exec -it centos6_1 bash
  [root@15f4870107f0 /]#
  
  ```

- `docker run --rm -it [image_name] bash`命令中的`--rm`选项可以让容器推出后直接删除，如果指定了运行的命令，则命令执行完容器就会被删除：

  ```bash
  [root@localhost ~]# docker run --rm -it centos bash -c "echo 123 && sleep 15"
  123
  
  [root@localhost ~]# docker ps -a
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                        PORTS               NAMES
  15f4870107f0        centos6             "bash"              4 minutes ago       Up 4 minutes                                      centos6_1
  04919c9dea63        centos              "bash"              16 minutes ago      Exited (127) 16 minutes ago                       hopeful_blackwell
  [root@localhost ~]#
  
  ```

- `docker logs [container_id]`命令可以获取到容器运行的历史信息，如容器的输出：

  ```bash
  [root@localhost ~]# docker run -itd centos bash -c "echo 123"
  5c1e603a1978fd8c018145ab8a39cb9eb22f6c93f26c5e36f3efb6ecf62e1154
  [root@localhost ~]# docker logs 5c1e603
  123
  [root@localhost ~]#
  
  ```

- `docker attach [container_id]`命令可以进入一个后台运行的容器，但使用该命令会使得用户在`exit`退出容器时，容器也停止运行，所以一般不推荐使用；

- `docker exec -it [container_id] bash`命令可以临时打开容器的虚拟终端，并且退出容器的终端后，容器依然保持运行；

- `docker rm [container_id]`可以删除停止运行的容器，使用`-f`选项则可以强制删除运行中的容器；

- `docker export [container_id] > file.tar`可以导出容器，这样可以将容器迁移到其他机器上导入；

- `cat file.tar | docker import - [image_name]`命令可以使用文件来导入镜像。

# 仓库管理

## 本地仓库推送镜像

- 我们使用`docker pull`和`docker push`拉取推送镜像时，都是对官方仓库拉取推送，对于企业环境来说，将项目使用的镜像推送到官方仓库是不合理的，所以docker官方提供了一个`registry`镜像，能够使用它来创建本地的docker私有仓库；

- 使用`docekr pull registry`拉取registry镜像：

  ```bash
  [root@localhost ~]# docker pull registry
  Using default tag: latest
  latest: Pulling from library/registry
  c87736221ed0: Pull complete
  1cc8e0bb44df: Pull complete
  54d33bcb37f5: Pull complete
  e8afc091c171: Pull complete
  b4541f6d3db6: Pull complete
  Digest: sha256:3b00e5438ebd8835bcfa7bf5246445a6b57b9a50473e89c02ecc8e575be3ebb5
  Status: Downloaded newer image for registry:latest
  
  ```

- `docker run -d -p 5000:5000 registry`命令启动registry镜像的容器，这里使用了`-p`选项，该选项可以将容器的端口映射到宿主机上，`:`左边是宿主机的端口，右边是容器的端口，如果不将容器的端口映射出来，那么只有本地宿主机才能通过容器的IP访问容器内的服务，通过映射，其他用户就可以访问宿主机的IP和映射出来的端口访问容器提供的服务：

  ```bash
  [root@localhost ~]# docker run -d -p 5000:5000 registry
  6f9aedc18569ec2b89127d88d5126c9b0ea18fa6d8c3822f1a9102aeeaedd1ba
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS    PORTS                    NAMES
  6f9aedc18569        registry            "/entrypoint.sh /e..."   3 seconds ago       Up 2 seconds    0.0.0.0:5000->5000/tcp   zealous_kilby
  
  [root@localhost ~]# telnet 127.0.0.1 5000
  Trying 127.0.0.1...
  Connected to 127.0.0.1.
  Escape character is '^]'.
  ^]
  telnet> ^CConnection closed.
  
  ```

- `curl 127.0.0.1:5000/v2/_catalog`可以访问registry容器，查看私有仓库内的镜像，默认为空：

  ```bash
  [root@localhost ~]# curl 127.0.0.1:5000/v2/_catalog
  {"repositories":[]}
  
  ```

- 接下来上传一个镜像到仓库中去，首先给镜像标记tag，tag必须带有私有仓库的`ip:port`:

  ```bash
  [root@localhost ~]# docker images
  REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
  centos6             latest              2f7d7ab8be8c        2 hours ago         553MB
  centos_with_net     latest              52992c2b5487        3 days ago          285MB
  centos              latest              9f38484d220f        3 weeks ago         202MB
  registry            latest              f32a97de94e1        4 weeks ago         25.8MB
  
  [root@localhost ~]# docker tag centos 192.168.139.128:5000/centos7
  
  [root@localhost ~]# docker images
  REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
  centos6                        latest              2f7d7ab8be8c        2 hours ago         553MB
  centos_with_net                latest              52992c2b5487        3 days ago          285MB
  192.168.139.128:5000/centos7   latest              9f38484d220f        3 weeks ago         202MB
  centos                         latest              9f38484d220f        3 weeks ago         202MB
  registry                       latest              f32a97de94e1        4 weeks ago         25.8MB
  
  ```

- 镜像标记完成之后，可以使用`docker push [image_name]`推送镜像到本地仓库，但是由于docker默认使用https上传镜像，所以直接push会提示失败：

  ```bash
  [root@localhost ~]# docker push 192.168.139.128:5000/centos7
  The push refers to a repository [192.168.139.128:5000/centos7]
  Get https://192.168.139.128:5000/v2/: http: server gave HTTP response to HTTPS client
  
  ```

- 解决上面的报错，需要修改docker的配置文件`/etc/docker/daemon.json`，在配置文件中添加下面的配置：

  ```json
  { "insecure-registries": ["192.168.139.128:5000"] }
  ```

- 然后重启docker服务并且重启容器，docker服务停止后，容器也会自动停止，所以需要重启容器：

  ```bash
  [root@localhost ~]# systemctl restart docker
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
  [root@localhost ~]# docker ps -a
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                 PORTS               NAMES
  6f9aedc18569        registry            "/entrypoint.sh /e..."   17 minutes ago      Exited (2) 7 seconds ago                             zealous_kilby
  5c1e603a1978        centos              "bash -c 'echo 123'"     About an hour ago   Exited (0) About an hour ago                         objective_roentgen
  15f4870107f0        centos6             "bash"                   About an hour ago   Exited (137) About an hour ago                       centos6_1
  [root@localhost ~]# docker start 6f9aedc18569
  6f9aedc18569
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS    PORTS                    NAMES
  6f9aedc18569        registry            "/entrypoint.sh /e..."   18 minutes ago      Up 3 seconds    0.0.0.0:5000->5000/tcp   zealous_kilby
  
  ```

- 完成后就可以使用`docker push [image_name]`推送镜像到本地仓库了：

  ```bash
  [root@localhost ~]# docker push 192.168.139.128:5000/centos7
  The push refers to a repository [192.168.139.128:5000/centos7]
  d69483a6face: Pushed
  latest: digest: sha256:ca58fe458b8d94bc6e3072f1cfbd334855858e05e1fd633aa07cf7f82b048e66 size: 529
  
  ```

- 最后查看本地仓库中的镜像是否上传成功：

  ```bash
  [root@localhost ~]# curl 192.168.139.128:5000/v2/_catalog
  {"repositories":["centos7"]}
  
  ```

  可以看到镜像centos7已经成功上传到本地仓库，这个镜像的名字就是我们给镜像标记的标签中ip地址后面的名字。

## 本地仓库拉取镜像

- 从私有仓库中拉取镜像，可以使用`docker pull [registry_ip:port/image_name]`命令:	

  ```bash
  [root@localhost ~]# docker pull 192.168.139.128:5000/centos7
  Using default tag: latest
  latest: Pulling from centos7
  Digest: sha256:ca58fe458b8d94bc6e3072f1cfbd334855858e05e1fd633aa07cf7f82b048e66
  Status: Downloaded newer image for 192.168.139.128:5000/centos7:latest
  [root@localhost ~]# docker images
  REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
  centos6                        latest              2f7d7ab8be8c        23 hours ago        553MB
  centos_with_net                latest              52992c2b5487        4 days ago          285MB
  192.168.139.128:5000/centos7   latest              9f38484d220f        3 weeks ago         202MB
  centos                         latest              9f38484d220f        3 weeks ago         202MB
  registry                       latest              f32a97de94e1        4 weeks ago         25.8MB
  
  ```

- 如果其他机器想要从私有仓库中拉取镜像，则必须在`/etc/docker/daemon.json`中写入如下配置：

  ```json
  { "insecure-registries":["192.168.139.128:5000"] }
  ```

- 然后使用上面的`docker pull`命令拉取镜像：

  ```bash
  [root@vm2 ~]# cat /etc/docker/daemon.json
  { "insecure-registries":["192.168.139.128:5000"] }
  
  [root@vm2 ~]# docker pull 192.168.139.128:5000/centos7
  Using default tag: latest
  latest: Pulling from centos7
  8ba884070f61: Pull complete
  Digest: sha256:ca58fe458b8d94bc6e3072f1cfbd334855858e05e1fd633aa07cf7f82b048e66
  Status: Downloaded newer image for 192.168.139.128:5000/centos7:latest
  
  ```

  

# 数据管理

## 挂载目录

- Docker的容器在运行之后被关闭或者删除，那么在容器运行时产生的数据，都会被一并删除，所以docker支持将宿主机的目录挂载到容器中去，这样容器产生的数据就会被写在宿主机上，即使容器被销毁，数据也不会丢失；

- 挂在宿主机本地的目录到容器中，使用`docker run -itd -v [host_dir]:[container_dir] [image_name] bash`命令，其中`:`左边是宿主机本地目录，右边是容器内的目录，指定的容器内的目录会自动被创建：

  ```bash
  [root@localhost ~]# docker run -itd -v /usr/local/src/:/data centos bash
  c470b7d4c11dcaf053c0729584b40c47e54ab651a5c197efda57bc38c804b903
  [root@localhost ~]# docker exec -it c470b7d4c1 bash
  [root@c470b7d4c11d /]# ls /data/
  nginx-1.12.2  nginx-1.12.2.tar.gz
  [root@c470b7d4c11d /]# exit
  [root@localhost ~]# ls /usr/local/src/
  nginx-1.12.2  nginx-1.12.2.tar.gz
  
  ```

- 在容器内挂载的目录，做的任何操作，都是对宿主机上的目录进行的操作。

## 挂载数据卷

- 在挂载目录创建容器的时候，是可以指定容器的`NAMES`的，如果没有指定，docker会自动给容器分配一个随机的`NAMES`，如下ID为c470b7d4c11d的容器的名字为`zealous_fermat`：

  ```bash
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
  c470b7d4c11d        centos              "bash"                   7 minutes ago       Up 7 minutes                                 zealous_fermat
  6f9aedc18569        registry            "/entrypoint.sh /e..."   22 hours ago        Up 21 hours         0.0.0.0:5000->5000/tcp   zealous_kilby
  
  ```

- 而docker在创建容器的时候可以使用`--volumes-from [container_name]`选项，来挂载其他容器的数据卷，命令为`docker run -itd --volumes-from [container_name] [image_name] bash`：

  ```bash
  [root@localhost ~]# docker run -itd --volumes-from zealous_fermat centos6 bash
  da1a932ea55df3db321cc164e1554bcfad5cd8a80078d4a2966aa9c4b2f59dd2
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
  da1a932ea55d        centos6             "bash"                   3 seconds ago       Up 2 seconds                                 nostalgic_boyd
  c470b7d4c11d        centos              "bash"                   20 minutes ago      Up 19 minutes                                zealous_fermat
  6f9aedc18569        registry            "/entrypoint.sh /e..."   22 hours ago        Up 22 hours         0.0.0.0:5000->5000/tcp   zealous_kilby
  [root@localhost ~]# docker exec -it da1a932ea55d bash
  [root@da1a932ea55d /]# ls /data/
  nginx-1.12.2  nginx-1.12.2.tar.gz
  
  ```

- 从上面执行的命令可以看到，`--volumes-form [container_name]`选项，就是将别的容器挂载的目录，当作数据卷挂载到新的容器，docker容器会自动识别挂载的目录，并在新的容器中挂载同样的目录，在新的容器中，同样可以对目录进行操作。

## 定义数据卷容器

- 实际上docker挂载目录的`-v`选项，可以不指定宿主机的目录，例如`docker run -itd -v /data/ --name testvol centos bash`这种形式，这样做的目的是创建一个数据卷容器，让该数据卷的目录能够被多个容器共享数据，类似于linux的NFS，其他容器可以直接使用`--volumes-from [container_name]`选项挂载该数据卷；

- 首先我们创建一个容器，并使用`-v`指定目录，**需要注意这里-v指定的目录，是容器里的目录，并非宿主机的目录**：

  ```bash
  [root@localhost ~]# docker run -itd -v /data/ --name testvol centos bash
  36f577a6cf197d99737f2d402cc0634ce450a95b33f2220994e7fc5567c63975
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
  36f577a6cf19        centos              "bash"                   3 seconds ago       Up 2 seconds                                 testvol
  da1a932ea55d        centos6             "bash"                   10 minutes ago      Up 10 minutes                                nostalgic_boyd
  c470b7d4c11d        centos              "bash"                   30 minutes ago      Up 30 minutes                                zealous_fermat
  6f9aedc18569        registry            "/entrypoint.sh /e..."   22 hours ago        Up 22 hours         0.0.0.0:5000->5000/tcp   zealous_kilby
  
  ```

- 然后再创建一个容器，并挂载数据卷：

  ```bash
  [root@localhost ~]# docker run -itd --volumes-from testvol centos6 bash
  2e75191c69e05732566f91644e0fdc4a8fbe57dfa30ac86bb6be891a4ef3fd87
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                    NAMES
  2e75191c69e0        centos6             "bash"                   6 seconds ago        Up 6 seconds                                 quirky_wozniak
  36f577a6cf19        centos              "bash"                   About a minute ago   Up About a minute                            testvol
  da1a932ea55d        centos6             "bash"                   12 minutes ago       Up 12 minutes                                nostalgic_boyd
  c470b7d4c11d        centos              "bash"                   32 minutes ago       Up 32 minutes                                zealous_fermat
  6f9aedc18569        registry            "/entrypoint.sh /e..."   22 hours ago         Up 22 hours         0.0.0.0:5000->5000/tcp   zealous_kilby
  [root@localhost ~]# docker exec -it 2e75191c69e0 bash
  [root@2e75191c69e0 /]# ls /data/
  
  ```

- 可以看到，数据卷容器分享的目录是什么，新的容器挂载的目录就是什么，两个容器已经共享了`/data`目录，如果有多个容器，同样也可以挂载数据卷访问数据卷的共享目录。

- 如果新建的容器，想要访问的共享目录并不是数据卷分享的目录名，而是其他目录名，那么可以通过创建软连接的形式来实现，例如，数据卷容器分享的目录是/data/，而新建的容器希望访问的是/home/目录，那么可以像下面这样实现：

  ```bash
  [root@localhost ~]# docker exec -it 2e75191c69e0 bash
  
  [root@2e75191c69e0 /]# mv /home home.1
  [root@2e75191c69e0 /]# ln -s /data/ /home
  [root@2e75191c69e0 /]# ls -l
  total 28
  dr-xr-xr-x.   2 root root 4096 Nov 26  2016 bin
  dr-xr-xr-x.   3 root root 4096 Nov 26  2016 boot
  drwxr-xr-x.   2 root root    6 Apr  9 10:01 data
  drwxr-xr-x.   5 root root  360 Apr  9 10:03 dev
  drwxr-xr-x.   1 root root   66 Apr  9 10:03 etc
  -rw-r--r--.   1 root root    0 Nov 26  2016 fastboot
  lrwxrwxrwx.   1 root root    6 Apr  9 10:11 home -> /data/
  drwxr-xr-x.   2 root root    6 Sep 23  2011 home.1
  dr-xr-xr-x.   8 root root   92 Nov 26  2016 lib
  dr-xr-xr-x.   7 root root 8192 Nov 26  2016 lib64
  drwx------.   2 root root    6 Nov 26  2016 lost+found
  drwxr-xr-x.   2 root root    6 Sep 23  2011 media
  drwxr-xr-x.   2 root root    6 Sep 23  2011 mnt
  drwxr-xr-x.   2 root root    6 Sep 23  2011 opt
  dr-xr-xr-x. 126 root root    0 Apr  9 10:03 proc
  dr-xr-x---.   2 root root   91 Nov 26  2016 root
  dr-xr-xr-x.   2 root root 4096 Nov 26  2016 sbin
  drwxr-xr-x.   2 root root    6 Sep 23  2011 selinux
  drwxr-xr-x.   2 root root    6 Sep 23  2011 srv
  dr-xr-xr-x.  13 root root    0 Mar 24 08:15 sys
  drwxrwxrwt.   2 root root    6 Nov 26  2016 tmp
  drwxr-xr-x.  13 root root  155 Nov 26  2016 usr
  drwxr-xr-x.  17 root root  197 Nov 26  2016 var
  
  ```

---