---
title: Docker部署Gitlab
author: Evobot
date: 2020-04-27 20:57:04
categories: 代码管理平台
tags:
  - Gitlab
  - Docker
image:
---



1. 使用docker创建gitlab容器
2. gitlab配置及优化

<!--more-->

---

# 创建docker容器

> 参考文档：[GitLab Docker映像安装]( https://docs.gitlab.com/omnibus/docker/ )

- 首先在宿主机上创建保存gitlab配置的目录：

  ```bash
  mkdir -p /data/gitlab/{config,logs,data}
  ```

- 创建环境变量，指向本地宿主机存储gitlab配置的目录：

  ```bash
  export GITLAB_HOME=/data/gitlab
  ```

- 然后从docker仓库拉取gitlab镜像：

  ```bash
  docker pull gitlab/gitlab-ce:latest
  ```

- 运行镜像，创建容器：

  ```bash
  docker run --detach \
  --hostname 192.168.139.128 \
  -p 443:443 -p 80:80 -p 1022:22 \
  --name gitlab --restart always \
  -v $GITLAB_HOME/config:/etc/gitlab \
  -v $GITLAB_HOME/logs:/var/log/gitlab \
  -v $GITLAB_HOME/data:/vat/opt/gitlab \
  gitlab/gitlab-ce:latest
  ```

  其中由于宿主机的22端口被sshd服务使用，所以改为映射1022端口。

# 配置gitlab

- 容器生成后，根据需要修改gitlab的配置，直接进入宿主机本地映射的config目录，编辑`gitlab.rb`文件;

- gitlab.rb文件内，主要需要配置的选项如下：

  ```erb
  ## GitLab NGINX
  nginx['listen_port'] = 80  # gitlab nginx 端口。默认端口为：80 
  
  ## GitLab Unicorn
  unicorn['listen'] = 'localhost'
  unicorn['port'] = 8080 #默认是8080端口
  
  ## GitLab URL 配置http协议所使用的访问地址
  external_url GENERATED_EXTERNAL_URL' # clone时显示的地址，gitlab 的域名
  
  # 配置ssh协议所使用的访问地址和端口
  gitlab_rails['gitlab_ssh_host'] = 'song.local'
  gitlab_rails['gitlab_shell_ssh_port'] = 10022
  ```

- gitlab性能相关的选项如下：

  ```erb
  # 超时时间
  unicorn['worker_timeout'] = 60        
                             
  #不能低于2，否则卡死 worker=CPU核数+1 
  unicorn['worker_processes'] = 2
  
  # 减少数据库缓存大小 默认256，可适当改小  
  postgresql['shared_buffers'] = "256MB"
  
  # 减少数据库并发数
  postgresql['max_worker_processes'] = 8
  
  # 减少sidekiq并发数
  sidekiq['concurrency'] = 10
  
  # 减少内存 
  unicorn['worker_memory_limit_min'] = "200 * 1 << 20"
  unicorn['worker_memory_limit_max'] = "300 * 1 << 20"
  ```

- 配置gitlab的邮箱服务：

  ```erb
  gitlab_rails['smtp_enable'] = true
  gitlab_rails['smtp_address'] = "smtp.server"
  gitlab_rails['smtp_port'] = 465
  gitlab_rails['smtp_user_name'] = "smtp user"
  gitlab_rails['smtp_password'] = "smtp password"
  gitlab_rails['smtp_domain'] = "example.com"
  gitlab_rails['smtp_authentication'] = "login"
  gitlab_rails['smtp_enable_starttls_auto'] = true
  gitlab_rails['smtp_openssl_verify_mode'] = 'peer'
  
  # If your SMTP server does not like the default 'From: gitlab@localhost' you
  # can change the 'From' with this setting.
  gitlab_rails['gitlab_email_from'] = 'gitlab@example.com'
  gitlab_rails['gitlab_email_reply_to'] = 'noreply@example.com'
  ```

- 配置完成后，重启gitlab容器即可。