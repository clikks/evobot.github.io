---
title: pyenv安装使用指南
author: Evobot
categories: Python
tags:
  - Python
  - Linux
abbrlink: 13c2c977
date: 2018-05-08 17:55:36
image:
---

在使用python的过程中，经常会遇到不同的项目可能使用的是不同的python版本，或者有些项目需要使用特定版本的软件包，为了保持系统python软件包不会变的混乱，我们可以使用pyenv来管理系统的python版本，并且使用virtualenv来管理不同的环境。
<!--more-->

------

# pyenv安装配置

- 官方的仓库和安装步骤可以查看[github-pyenv](https://github.com/pyenv/pyenv)，为了避免安装时报错，首先在shell中声明全局变量：

  ```bash
  $ export PYTHON_CONFIGURE_OPTS="--enable-shared"	
  ```

## 克隆仓库

- 克隆最新版本的pyenv仓库，这里克隆到家目录下，也可以自行指定克隆到其他目录：

  ```bash
  $ git clone git://github.com/yyuu/pyenv.git ~/.pyenv
  ```

## 配置pyenv

1. 为pyenv添加环境变量，如果先前克隆仓库时指定了其他目录，这里也要相应的更改pyenv的目录：

   ```bash
   $ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
   $ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
   ```

   {% note info %}

   使用zsh的需要将上面的`~/.bash_profile`替换为`~/.zshenv`；

   Ubuntu和Fedora则使用`~/.bashrc`替换；

   {% endnote %}

2. 然后将pyenv的初始化命令添加到shell中：

   ```bash
   $ echo -e 'if command -v pyenv 1>/dev/null 2>&1; then\n  eval "$(pyenv init -)"\nfi' >> ~/.bash_profile
   ```

   {% note info %}

   使用zsh的需要将上面的`~/.bash_profile`替换为`~/.zshenv`；

   Ubuntu和Fedora则使用`~/.bashrc`替换；

   {% endnote %}

3. 安装完成后，需要重新登陆shell或者重新载入配置文件，如果上面替换过`.bash_profile`文件，则下面的命令也要更改为替换的文件名：

   ```bash
   $ source ~/.bash_pofile
   ```

4. 执行命令`pyenv --version`或`pyenv --help`来验证pyenv安装是否正确:

   ```bash
   $ pyenv --version
   pyenv 1.2.4
   $ pyenv --help
   Usage: pyenv <command> [<args>]

   Some useful pyenv commands are:
      commands    List all available pyenv commands
      local       Set or show the local application-specific Python version
      global      Set or show the global Python version
      shell       Set or show the shell-specific Python version
      install     Install a Python version using python-build
      uninstall   Uninstall a specific Python version
      rehash      Rehash pyenv shims (run this after installing executables)
      version     Show the current Python version and its origin
      versions    List all Python versions available to pyenv
      which       Display the full path to an executable
      whence      List all Python versions that contain the given executable

   See `pyenv help <command>' for information on a specific command.
   For full documentation, see: https://github.com/pyenv/pyenv#readme
   ```

## 更换国内源

- 由于使用pyenv默认会到官网下载python版本，导致下载速率非常缓慢，所以可以使用国内的源来加速pyenv的版本安装速度，pyenv的python版本下载配置文件每个都是单独的，所以需要针对要下载的版本修改其配置文件；

- pyenv的python版本配置文件在目录`~/.pyenv/plugins/python-build/share/python-build/`下，例如需要下载python3.6.0版本，这里替换为搜狐的源<http://mirrors.sohu.com/python/>。

- 修改`3.6.0`文件如下：

  ```bash
  #require_gcc
  install_package "openssl-1.0.2k" "https://www.openssl.org/source/openssl-1.0.2k.tar.gz#6b3977c61f2aedf0f96367dcfb5c6e578cf37e7b8d913b4ecb6643c3cb88d8c0" mac_openssl --if has_broken_mac_openssl
  install_package "readline-6.3" "https://ftpmirror.gnu.org/readline/readline-6.3.tar.gz#56ba6071b9462f980c5a72ab0023893b65ba6debb4eeb475d7a563dc65cafd43" standard --if has_broken_mac_readline
  if has_tar_xz_support; then
    #install_package "Python-3.6.0" "https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tar.xz#b0c5f904f685e32d9232f7bdcbece9819a892929063b6e385414ad2dd6a23622" ldflags_dirs standard verify_py36 ensurepip
    # 注释原下载地址，增加下面的搜狐源下载地址
    install_package "Python-3.6.0" "http://mirrors.sohu.com/python/3.6.0/Python-3.6.0.tar.xz" ldflags_dirs standard verify_py36 ensurepip
  else
    #install_package "Python-3.6.0" "https://www.python.org/ftp/python/3.6.0/Python-3.6.0.tgz#aa472515800d25a3739833f76ca3735d9f4b2fe77c3cb21f69275e0cce30cb2b" ldflags_dirs standard verify_py36 ensurepip
    # 同样进行替换
    install_package "Python-3.6.0" "http://mirrors.sohu.com/python/3.6.0/Python-3.6.0.tgz" ldflags_dirs standard verify_py36 ensurepip
  fi
  ```

- 修改完成后，执行`pyenv install 3.6.0`就可以看到下载是从搜狐的源进行下载，下载完成的python包保存在`~/.pyenv/cache`下：

  ```bash
  $ pyenv install 3.6.0
  Downloading Python-3.6.0.tar.xz...
  -> http://mirrors.sohu.com/python/3.6.0/Python-3.6.0.tar.xz
  Installing Python-3.6.0...
  Installed Python-3.6.0 to /home/evobot/.pyenv/versions/3.6.0
  ```

- 正是因为下载的包在`~/.pyenv/cache`目录下，所以我们也可以使用另一种方法来加速下载python版本包，直接到[搜狐源](http://mirrors.sohu.com/python/)下载需要的版本，然后放入`~/.pyenv/cache`目录，再执行`pyenv install`安装即可，也可以使用下面的一建执行命令：

  ```bash
  v=3.5.2|wget http://mirrors.sohu.com/python/$v/Python-$v.tar.xz -P ~/.pyenv/cache/;pyenv install $v
  ```

  > 其中变量v对应要下载的python版本号。

# pyenv使用

- 想要查看pyenv支持哪些python版本，可以执行`pyenv install --list`命令查看：

  ```bash
  $ pyenv install --list
  Available versions:
    2.1.3
    2.2.3
    2.3.7
    ...
  ```

- 安装所需python版本命令如下，其中`-v`选项表示安装时是否输出详细信息：

  ```bash
  $ pyenv install -v 3.6.0
  $ pyenv install 3.6.0
  ```

- `pyenv versions`查看当前系统已安装的python版本，其中有`*`的表示当前生效的版本，而`pyenv version`则是查看当前生效的python版本：

  ```bash
  $ pyenv versions
  * system (set by /root/.pyenv/version)
    3.5.0
  $ pyenv version
  3.5.0 (set by /root/.pyenv/version)
  ```

- 卸载python版本命令如下：

  ```bash
  $ pyenv versions
  * system (set by /root/.pyenv/version)
    3.5.0
    
  $ pyenv uninstall 3.5.0
  pyenv: remove /root/.pyenv/versions/3.5.0? y

  $ pyenv versions
  * system (set by /root/.pyenv/version)
  ```

- 初始`pyenv versions`只有**system**，即系统全局的python版本，pyenv可以针对系统全局、目录、shell分别设置不同的python版本：

  ```bash
  # 设置全局python版本，版本号将会写入~/.pyenv/version
  $ pyenv global 3.5.0

  # 为当前目录设置python版本，版本号写入当前目录的.python-version文件。
  # local设置的python版本优先级比global高。
  $ pyenv local 3.6.0
  $ pyenv versions
    system
    3.5.0
  * 3.6.0 (set by /root/code/.python-version)

  # 设置当前shell的python版本是通过设置PYENV_VERSION环境变量改变的。
  # shell设置的pthon版本优先级比global和local都要高。
  $ pyenv shell 3.5.0
  $ pyenv versions
    system
  * 3.5.0 (set by PYENV_VERSION environment variable)
    3.6.0
    
   # 取消当前shell的python版本设置
   $ pyenv shell --unset
  ```

- 在每次增删python版本或者pip安装了新的包后，都需要执行`pyenv rehash`更新shims。

> 更多的pyenv命令，可以查看[官方文档](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md#command-reference)。

# virtualenv插件

pyenv的virtualenv插件，能够实现与virtualenv相同的功能，使用pyenv管理系统python版本，使用virtualenv管理不同的python环境，从而实现不同的项目在相同的python版本时，也能够同时使用各自的python环境。

## 安装

- [pyenv-virtualenv](https://github.com/pyenv/pyenv-virtualenv)插件安装直接使用git克隆到pyenv的Plugins目录：

  ```bash
  $ git clone https://github.com/pyenv/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
  ```

- 添加命令到`~/.bash_profile`，zsh用户添加到`~/.zshenv`，Ubuntu用户添加到`~/.bashrc`，完成后更新shell或重新登陆生效：

  ```bash
  $ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
  $ exec "$SHELL"
  ```

## 使用

- pyenv-virtualenv可以为同一个python版本创建多个虚拟工作环境，命令为`pyenv virtualenv [python版本号] [虚拟环境名]`，为当前的python版本创建虚拟环境的命令为`pyenv virtualenv [虚拟环境名]`：

  ```bash
  $ pyenv virtualenv 3.5.0 my-virtual-env-3.5.0

  $ pyenv version
  3.4.3 (set by /home/yyuu/.pyenv/version)
  $ pyenv virtualenv venv34
  ```

- 使用`pyenv virtualenvs`查看系统当前存在的虚拟环境：

  ```bash
  $ pyenv virtualenvs
    3.5.0/envs/venv35 (created from /root/.pyenv/versions/3.5.0)
    venv35 (created from /root/.pyenv/versions/3.5.0)
  ```

- 如果在目录中使用`pyenv local [虚拟环境名]`为目录设置工作环境，那么在进入和离开目录时，会自动激活和去激活工作环境，并且在进入目录时，会在命令行开头显示当前的工作环境：

  ```bash
  root@ubuntu:~/code# pyenv local venv35
  (venv35) root@ubuntu:~/code#
  ```

- 如果没有使用`pyenv local`命令为目录设置工作环境的话，也可以使用`pyenv activate [虚拟环境名]`来临时激活一个全局的工作环境，去激活使用`pyenv deactivate`命令：

  ```bash
  root@ubuntu:~/code2# pyenv activate venv35
  pyenv-virtualenv: prompt changing will be removed from future release. configure `export PYENV_VIRTUALENV_DISABLE_PROMPT=1' to simulate the behavior.
  (venv35) root@ubuntu:~/code2# cd ..
  (venv35) root@ubuntu:~#

  (venv35) root@ubuntu:~# pyenv deactivate
  root@ubuntu:~#
  ```

- 删除已有的虚拟环境使用`pyenv uninstall my-virtual-env`命令：

  ```bash
  # pyenv uninstall venv35
  pyenv-virtualenv: remove /root/.pyenv/versions/3.5.0/envs/venv35? y
  ```

# pip更换源

- pip安装python包经常也会非常缓慢，可以将其更换为国内源，在家目录下创建`.pip`目录，并创建`.pip/pip.conf`文件，文件内容如下：

  ```bash
  [global]
  index-url = http://mirrors.aliyun.com/pypi/simple/
  [install]
  trusted-host=mirrors.aliyun.com
  ```

- 这里使用阿里的pip镜像，使用`pip install`可以看到已经变成从阿里的镜像下载python包：

  ```bash
  $ pip install plumbum
  Collecting plumbum
    Downloading http://mirrors.aliyun.com/pypi/packages/b2/05/7720109462d0bd60466e74076a38ca12068771da146bfd18a502726c9da8/plumbum-1.6.6-py2.py3-none-any.whl (111kB)
      100% |████████████████████████████████| 112kB 1.2MB/s
  Installing collected packages: plumbum
  Successfully installed plumbum-1.6.6
  ```

------

