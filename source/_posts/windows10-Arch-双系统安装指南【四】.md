---
title: windows10 & Arch 双系统安装指南【四】
tags:
  - archlinux
  - cinnamon
  - windows10
categories: Linux
abbrlink: f5f3d6e0
date: 2016-02-18 22:05:11
---
![](https://s1.ax1x.com/2020/04/28/JI8XE8.png)
经过前一篇的步骤，我们完成了常用的应用软件的安装，可以说现在arch已经完全不影响使用了，但是，我们都希望自己的系统环境比较美观，所以，这一篇，我们来对系统进行美化，具体包括主题安装，扩展设置，字体配置等等。
<!-- more -->

---
# 字体配置
## 字体安装
　　我们之前安装了文泉驿字体，我平时使用的还有ubuntu字体，所以我们再安装一个ubuntu字体包：
```bash
$ sudo pacman -S ttf-ubuntu-font-family
```
## 字体配置
　　现在即使切换到中文，我们也会发现整个系统的字体发虚，所以我们需要对系统的字体配置作一些改动：
　　[点击此处](http://pan.baidu.com/s/1bnYDy8J)下载字体配置文件。
```bash
#备份系统原来的字体配置文件
$ sudo mv /etc/fonts /etc/fonts.bak
#将下载的字体配置解压过去
$ sudo tar zxvf fonts.tar.gz -C /etc/
```
## 安装gnome优化工具
```bash
sudo pacman -S gnome-tweak-tool
```
打开优化工具，将字体设置为自己喜欢的，下面是我的配置：
![](https://s1.ax1x.com/2020/04/28/JI8v4g.png)
在系统的字体设置里面也使用同样的配置。

# 主题配置
## 安装numix图标
```bash
$ sudo pacman -S numix-circle-icon-theme-git
```
## 安装主题
[点击此处](http://pan.baidu.com/s/1hrwjM3y)下载Yosemite主题
```bash
#将主题解压到系统目录
$ sudo tar zxvf Yosemite Light.0.3.tar.gz -C /usr/share/themes
```
## 安装鼠标主题
[点击此处](http://pan.baidu.com/s/1jHuJTZ4)下载鼠标主题
```bash
#解压鼠标主题到系统目录
tar zxvf OpenZone.tar.gz -C /usr/share/icons
```
现在我们的主题就全部安装完成了，剩下的只需要在系统设置里的主题里设置一下就可以了，在主题设置里，下载Numix桌面主题，下面是我的配置：
![](https://s1.ax1x.com/2020/04/28/JI8zCQ.png)

# VIM配置
## 命令别名
设置命令vi为vim
```bash
vi ～/.bashrc
#添加一行：
alias vi='vim'
#编辑vimrc文件
vi ~/.vimrc
```
在.vimrc中写入以下内容
> syntax on
set tabstop=4
filetype on
set nocp
set mouse=a
set selection=exclusive
set selectmode=mouse,key
set showcmd
set laststatus=2
set ruler
set ignorecase
set hlsearch
set incsearch
filetype plugin on
filetype plugin indent on
set completeopt=longest,menu
set wildmenu
autocmd FileType ruby,eruby set omnifunc=rubycomplete#Complete
autocmd FileType python set omnifunc=pythoncomplete#Complete
autocmd FileType javascript set omnifunc=javascriptcomplete#CompleteJS
autocmd FileType html set omnifunc=htmlcomplete#CompleteTags
autocmd FileType css set omnifunc=csscomplete#CompleteCSS
autocmd FileType xml set omnifunc=xmlcomplete#CompleteTags
autocmd FileType java set omnifunc=javacomplete#Complet

为了让我们在使用`sudo vi`命令时的vi也能调用vim，我们还需要在**.bashrc**里添加一个命令别名：`alias sudo='sudo '`，注意，后面的sudo之后有一个空格
# lightdm美化
安装lightdm-gtk-greeter-settings
```bash
$ sudo pacman -S lightdm-gtk-greeter-settings
```
打开lightdm-gtk-greeter-settings，可以配置登陆界面的背景图片，头像等。
# 其他配置
安装docky:
```bash
$ sudo pacman -S docky
```
最后换个壁纸，按照自己的喜好配置一下，就可以了。
送上壁纸:
![](https://s1.ax1x.com/2020/04/28/JIGPuq.jpg)