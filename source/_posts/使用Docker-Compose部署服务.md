---
title: 使用Docker Compose部署服务
author: Evobot
categories: 虚拟化与容器化
tags:
  - Docker
abbrlink: 31ddd77e
date: 2019-04-14 01:16:12
image:
---

1. 用Docker Compose部署服务
2. Docker Compose示例

<!--more-->

---

# Docker Compose

## 介绍

- Docker Compose可以方便我们快捷高效的管理容器的启动、停止、重启等操作，例如环境中存在nginx、redis、mysql等多个镜像容器，如果要对这些容器进行管理，就需要一个个的去启动、停止等，非常繁琐，而Docker Compose就是解决这个问题的；
- Docker Compose类似于linux下的shell脚本，基于yaml语法，在该文件里，我们可以描述应用的架构，比如使用什么镜像、数据卷、网络模式、监听端口等信息；
- 我们可以在一个compose文件中定义一个多容器的应用，例如jumpserver，然后通过该compose来启动这个应用。

## 安装compose

- 使用以下命令安装当前最新的稳定版Docker Compose：

  ```bash
  sudo curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
  ```

- 然后为`docker-compose`命令赋予执行权限：

  ```bash
  sudo chmod +x /usr/local/bin/docker-compose
  ```

- 完成后，可以使用`docker-compose version`命令查看版本号：

  ```bash
  [root@localhost ~]# docker-compose version
  docker-compose version 1.24.0, build 0aa59064
  docker-py version: 3.7.2
  CPython version: 3.6.8
  OpenSSL version: OpenSSL 1.1.0j  20 Nov 2018
  
  ```

- docker compose区分Version1和Version2（Compose 1.6.0+，Docker Engine 1.10.0+），Version2支持更多的指令，在compose的yaml文件里没有声明时，默认是Version1版本，Version1将来会被弃用。

# Docker Compose示例

- 首先创建一个`docker-compose.yml`文件，写入文件内容如下：

  ```yaml
  # version指定compose版本
  version: "2"
  # services指定容器或镜像需要做的操作
  services:
    # app1表示容器名
    app1:
      # 指定容器的镜像
      image: centos_nginx
      # 指定容器要映射的端口
      ports:
        - "8080:80"
      # networks指定容器使用的网络，网络的具体定义在后面的networks内定义，不指定默认为bridge
      networks:
        - "net1"
      # volumes指定容器要挂载的目录
      volumes:
        - /data/:/data
    app2:
      image: centos_with_net
      networks:
        - "net2"
      volumes:
        - /data/:/data1
      # 指定容器启动时运行的命令，如果镜像在制作时指定了entrypoint，这里可以不指定entrypoint，否则容器启动后就会立即停止
      entrypoint: tail -f /etc/passwd
  networks:
    net1:
      driver: bridge
    net2:
      driver: bridge
  
  ```

- 在存放docker-compose.yml文件的目录内，使用`docker-compose up -d`运行docker-compose，其中`-d`是让容器后台启动，否则容器运行时的输出都是打印到前台：

  ```bash
  [root@localhost ~]# docker-compose up -d
  Creating network "root_net1" with driver "bridge"
  Creating network "root_net2" with driver "bridge"
  Creating root_app2_1 ... done
  Creating root_app1_1 ... done
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
  49210131e9c3        centos_with_net     "tail -f /etc/passwd"    6 seconds ago       Up 3 seconds                               root_app2_1
  0d12af583edf        centos_nginx        "/bin/sh -c '/usr/..."   6 seconds ago       Up 4 seconds        0.0.0.0:8080->80/tcp   root_app1_1
  
  ```

  可以看到，指定的app1和app2容器已经被批量启动。

- 停止docker-compose定义的容器，使用`docker-compose stop`命令，同样要在docker-compose.yml同级目录内执行：

  ```bash
  [root@localhost ~]# docker-compose stop
  Stopping root_app2_1 ... done
  Stopping root_app1_1 ... done
  [root@localhost ~]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
  7c6962078906        centos_nginx        "/bin/sh -c '/usr/..."   45 hours ago        Up 45 hours         0.0.0.0:81->80/tcp   nginx
  
  ```

- 查看`docker-compose`命令支持的选项，使用`docker-compose -h`命令：

  ```bash
  [root@localhost ~]# docker-compose -h
  Define and run multi-container applications with Docker.
  
  Usage:
    docker-compose [-f <arg>...] [options] [COMMAND] [ARGS...]
    docker-compose -h|--help
  
  Options:
    -f, --file FILE             Specify an alternate compose file
                                (default: docker-compose.yml)
    -p, --project-name NAME     Specify an alternate project name
                                (default: directory name)
    --verbose                   Show more output
    --log-level LEVEL           Set log level (DEBUG, INFO, WARNING, ERROR, CRITICAL)
    --no-ansi                   Do not print ANSI control characters
    -v, --version               Print version and exit
    -H, --host HOST             Daemon socket to connect to
  
    --tls                       Use TLS; implied by --tlsverify
    --tlscacert CA_PATH         Trust certs signed only by this CA
    --tlscert CLIENT_CERT_PATH  Path to TLS certificate file
    --tlskey TLS_KEY_PATH       Path to TLS key file
    --tlsverify                 Use TLS and verify the remote
    --skip-hostname-check       Don't check the daemon's hostname against the
                                name specified in the client certificate
    --project-directory PATH    Specify an alternate working directory
                                (default: the path of the Compose file)
    --compatibility             If set, Compose will attempt to convert keys
                                in v3 files to their non-Swarm equivalent
  
  Commands:
    build              Build or rebuild services
    bundle             Generate a Docker bundle from the Compose file
    config             Validate and view the Compose file
    create             Create services
    down               Stop and remove containers, networks, images, and volumes
    events             Receive real time events from containers
    exec               Execute a command in a running container
    help               Get help on a command
    images             List images
    kill               Kill containers
    logs               View output from containers
    pause              Pause services
    port               Print the public port for a port binding
    ps                 List containers
    pull               Pull service images
    push               Push service images
    restart            Restart services
    rm                 Remove stopped containers
    run                Run a one-off command
    scale              Set number of containers for a service
    start              Start services
    stop               Stop services
    top                Display the running processes
    unpause            Unpause services
    up                 Create and start containers
    version            Show the Docker-Compose version information
  
  ```

- `docker-compose ps`可以查看当前目录内定义的docker-compose容器状态：

  ```bash
  [root@localhost ~]# docker-compose ps
     Name                  Command                State     Ports
  ---------------------------------------------------------------
  root_app1_1   /bin/sh -c /usr/local/ngin ...   Exit 0
  root_app2_1   tail -f /etc/passwd              Exit 137
  
  #使用-f选项查看指定docker-compose配置的容器
  [root@localhost /]# docker-compose -f /root/docker-compose.yml ps
     Name                  Command                State     Ports
  ---------------------------------------------------------------
  root_app1_1   /bin/sh -c /usr/local/ngin ...   Exit 0
  root_app2_1   tail -f /etc/passwd              Exit 137
  
  ```

---

