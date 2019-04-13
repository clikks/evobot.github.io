---
title: 使用Dockerfile创建镜像
author: Evobot
date: 2019-04-13 21:22:51
categories: 虚拟化与容器化
tags:
  - Docker
image:
---

1. Dockerfile格式
2. Dockerfile示例（安装nginx）

<!--more-->
---

# Dockerfile格式

- 前面我们创建镜像可以通过容器导出为镜像，也可以下载模板导入为镜像，而Dockerfile是由一系列命令和参数构成的脚本，这些命令应用于基础镜像并最终创建一个新的镜像，Dockerfile能够简化镜像的创建流程并简化部署工作；

- Dockerfile的格式主要包括一下几个主要参数：

  - `FROM`：用于指定基于哪个基础镜像，格式如下

  ```dockerfile
  FROM <image>
  或者
  FROM <image>:<tag>
  ```

  ```dockerfile
  FROM centos
  or
  FROM centos:latest
  ```

  - `MAINTAINER`：用于指定作者信息，该选项为可选，可以有也可以没有。格式如下

  ```dockerfile
  MAINTAINER <author_name>
  ```

  ```dockerfile
  MAINTAINER evobot evobot@evobot.cn
  ```

  - `RUN`：镜像操作指令，镜像内需要安装软件包或者执行命令等操作，就需要使用`RUN`指令，格式如下

  ```dockerfile
  RUN <command>
  or
  RUN ["executable","param1","param2"]
  ```

  ```dockerfile
  RUN yum install httpd
  or
  RUN ["/bin/bash","-c","echo hello world"]
  ```

  - `CMD`：用来指定容器启动时用到的命令，**只能有一条**，格式如下：

  ```dockerfile
  CMD ["executable","param1","param2"]
  or
  CMD command param1 param2
  or
  CMD ["param1","param2"]
  ```

  ```dockerfile
  CMD ["/bin/bash","/usr/local/nginx/sbin/nginx","-c","/usr/local/nginx/conf/nginx.conf]
  ```

  - `EXPOSE`：指定要暴露出去的端口，比如容器内部启动了sshd和nginx，需要将22和80端口暴露出去，使得容器启动时能够映射对应的端口，这个选项需要配合`docker run `的`-P`或者`-p`选项来工作，`-P`是将端口映射到宿主机上自动分配的端口上，使用时后面不用指定容器的端口，会自动将`EXPOSE`暴露的端口映射出来， `-p`则需要指定要映射到宿主机上具体端口号，格式如下：

  ```dockerfile
  EXPOSE <port1> <port2>
  ```

  ```dockerfile
  EXPOSE 22 80 443
  ```

  - `ENV`：为后续的RUN指令提供环境变量，另外也可以自定义变量，格式如下：

  ```dockerfile
  ENV <key> <value>
  ```

  ```dockerfile
  ENV PATH /usr/local/mysql/bin:$PATH
  ENV MYSQL_version 5.6
  ```

  - `ADD`：将本地的文件或者目录拷贝到容器的某个目录里，其中src为Dockerfile所在目录的相对路径，src也可以是一个url，格式如下：

  ```dockerfile
  ADD <src> <dest>
  ```

  ```dockerfile
  ADD <conf/vhosts> </usr/local/nginx/conf>
  ```

  - `COPY`：格式与`ADD`相同，但是不支持url，同样是将文件或目录拷贝到容器里；
  - `ENTRYOPOINT`：容器启动时要执行的命令，格式和`CMD`类似，也是只有一条生效，写多条只有最后一条生效，和`CMD`不同的是`CMD`指定的命令是可以被docker run指令覆盖的，而`ENTRYPOINT`不能覆盖，例如在Dockerfile中指定CMD为`CMD ["/bin/echo","test"]`，若启动容器的命令是`docker run centos`，则会输出test，若启动容器的命令是`docker run -it centos /bin/bash`，则不会输出test，而ENTRYPOINT不会被覆盖，而且会比CMD或者docker run指定的命令要优先执行，比如Dockerfile中ENTRYPOINT为`ENTRYPOINT [”echo","test"]`，那么容器启动时`docker run -it centos 123`，则会输出test 123，相当于执行了`echo test 123`；
  - `VOLUME`： 格式为`VOLUME ["/data"]`，创建一个可以从本地主机或其他容器挂载的挂载点，通过VOLUME创建的挂载点，无法指定宿主机上对应的目录，是docker自动生成的，而查看docker生成的目录位置，使用`docker inspect [container_id] `查看Mounts项的数据;
  - `USER`：指定运行容器的用户，格式为`USER username`；
  - `WORKDIR`：为后续的RUN、CMD、ENTRYPOINT指定工作目录，格式为`WORKDIR /path/to/workdir`，指定了WORKDIR后，RUN、CMD、ENTRYPOINT就会在WORKDIR指定的目录下执行；

# Dockerfile安装nginx

- 本节创建一个安装nginx的Dockerfile，首先创建一个`nginx_dockerfile`目录，进入目录后，创建`Dockefile`文件：

  ```bash
  [root@localhost ~]# mkdir nginx_dockerfile
  [root@localhost ~]# cd nginx_dockerfile/
  [root@localhost nginx_dockerfile]# touch Dockerfile
  
  ```

- 然后编辑Dockerfile文件，内容如下：

  ```dockerfile
  # Set the base image to CentOS
  FROM centos
  # File Author /Maintainer
  MAINTAINER evobot www.evobot.cn
  # Install necessary tools
  RUN yum install -y pcre-devel wget net-tools gcc zlib zlib-devel make openssl-devel
  # Install Nginx
  ADD http://nginx.org/download/nginx-1.15.9.tar.gz .
  RUN tar zxvf nginx-1.15.9.tar.gz
  RUN mkdir -p /usr/local/nginx
  RUN cd nginx-1.15.9 && ./configure --prefix=/usr/local/nginx && make && make install
  RUN rm -fv /usr/local/nginx/conf/nginx.conf
  COPY nginx_conf /usr/local/nginx/conf/nginx.conf
  # Expose ports
  EXPOSE 80
  # Set the default command to execute when creating a new container
  ENTRYPOINT /usr/local/nginx/sbin/nginx -g 'daemon off;'
  
  ```

  这里ENTRYPOINT里的nginx启动命令后面加了`-g 'daemon off;'`是因为如果不加这句，容器启动后就会停止，无法后台运行，这是因为Docker 容器启动时，默认会把容器内部第一个进程，也就是`pid=1`的程序，作为docker容器是否正在运行的依据，如果 docker 容器pid=1的进程挂了，那么docker容器便会直接退出。

  Docker未执行自定义的CMD之前，nginx的pid是1，执行到CMD之后，nginx就在后台运行，bash或sh脚本的pid变成了1。所以一旦执行完自定义CMD，nginx容器也就退出了；

- 将nginx_conf配置文件拷贝到nginx_dockerfile目录下：

  ```bash
  [root@localhost nginx_dockerfile]# cp /usr/local/nginx/conf/nginx.conf nginx_conf
  ```

- 接着使用`docker build -t [image_name] .`命令构建镜像，docker构建镜像时会将每一步的执行输出到前台：

  ```bash
  [root@localhost nginx_dockerfile]# docker build -t centos_nginx .\
  >
  Sending build context to Docker daemon   5.12kB
  Step 1/11 : FROM centos
   ---> 9f38484d220f
  Step 2/11 : MAINTAINER evobot www.evobot.cn
   ---> Using cache
   ---> 36180a65d0ee
  Step 3/11 : RUN yum install -y pcre-devel wget net-tools gcc zlib zlib-devel make openssl-devel
   ---> Using cache
   ---> 5a4d711115c8
  Step 4/11 : ADD http://nginx.org/download/nginx-1.15.9.tar.gz .
  Downloading [==================================================>]  1.032MB/1.032MB
   ---> Using cache
   ---> bdf5292f9e42
  Step 5/11 : RUN tar zxvf nginx-1.15.9.tar.gz
   ---> Using cache
   ---> b1ffc479c8bb
  Step 6/11 : RUN mkdir -p /usr/local/nginx
   ---> Using cache
   ---> 50fac2d5b701
  Step 7/11 : RUN cd nginx-1.15.9 && ./configure --prefix=/usr/local/nginx && make && make install
   ---> Using cache
   ---> 6a21a404ec3a
  Step 8/11 : RUN rm -fv /usr/local/nginx/conf/nginx.conf
   ---> Using cache
   ---> 1cce78290a80
  Step 9/11 : COPY nginx_conf /usr/local/nginx/conf/nginx.conf
   ---> 07d4ad0ee2f6
  Step 10/11 : EXPOSE 80
   ---> Running in 59909124c7a7
   ---> 33fdb3c11e9e
  Removing intermediate container 59909124c7a7
  Step 11/11 : ENTRYPOINT /usr/local/nginx/sbin/nginx
   ---> Running in c9886dbe001e
   ---> 5b903414e2cc
  Removing intermediate container c9886dbe001e
  Successfully built 5b903414e2cc
  Successfully tagged centos_nginx:latest
  
  ```

- 完成后查看镜像：

  ```bash
  [root@localhost nginx_dockerfile]# docker images
  REPOSITORY                     TAG                 IMAGE ID            CREATED             SIZE
  centos_nginx                   latest              5b903414e2cc        4 minutes ago       389MB
  centos                         latest              9f38484d220f        4 weeks ago         202MB
  
  ```

- 然后可以尝试使用镜像创建容器看看：

  ```bash
  [root@localhost nginx_dockerfile]# docker run -itd -p 81:80 --name nginx centos_nginx bash
  7c6962078906d15cef09382b8762483e717310d6fae7c3d17f5608ace5ab55f5
  [root@localhost nginx_dockerfile]# docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                NAMES
  7c6962078906        centos_nginx        "/bin/sh -c '/usr/..."   3 seconds ago       Up 2 seconds        0.0.0.0:81->80/tcp   nginx
  
  ```

- 进入容器，查看nginx的进程：

  ```bash
  [root@localhost nginx_dockerfile]# docker exec -it nginx bash
  [root@7c6962078906 /]# ps aux
  USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root          1  0.0  0.3  20556  1648 pts/0    Ss+  16:41   0:00 nginx: master process /usr/local/nginx/sbin/nginx -g daemon off;
  nobody        5  0.0  0.6  23044  3220 pts/0    S+   16:41   0:00 nginx: worker process
  nobody        6  0.0  0.6  23044  3220 pts/0    S+   16:41   0:00 nginx: worker process
  root          7  0.3  0.3  11796  1888 pts/1    Ss   16:43   0:00 bash
  root         19  0.0  0.3  51740  1744 pts/1    R+   16:43   0:00 ps aux
  [root@7c6962078906 /]# netstat -tlnp|grep 80
  tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1/nginx: master pro
  
  ```

- 宿主机访问81端口的nginx服务：

  ```html
  [root@localhost nginx_dockerfile]# curl localhost:81
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  <style>
      body {
          width: 35em;
          margin: 0 auto;
          font-family: Tahoma, Verdana, Arial, sans-serif;
      }
  </style>
  </head>
  <body>
  <h1>Welcome to nginx!</h1>
  <p>If you see this page, the nginx web server is successfully installed and
  working. Further configuration is required.</p>
  
  <p>For online documentation and support please refer to
  <a href="http://nginx.org/">nginx.org</a>.<br/>
  Commercial support is available at
  <a href="http://nginx.com/">nginx.com</a>.</p>
  
  <p><em>Thank you for using nginx.</em></p>
  </body>
  </html>
  
  ```

  服务正常，通过Dockerfile创建镜像并启动容器的nginx就成功了。

---