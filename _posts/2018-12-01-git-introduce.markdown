---
layout:       post
title:        "git安装及使用"
subtitle:     "git instructions"
date:         2018-12-01 22:05:00
author:       "ZeFeng"
header-img:   "img/ice_header_image.jpg"
header-mask:  0.3
catalog:      true
tags:
    - git搭建及使用
---

## 简介
&nbsp;&nbsp;&nbsp;&nbsp;git是目前最流行的分布式版本控制系统，分布式版本控制系统的安全性相对集中式来说更高，因为每个人电脑里都有完整的版本库，某一个人的电脑坏掉了不要紧，随便从其他人那里复制一个就可以了。<br />

## Linux上安装Git
&nbsp;&nbsp;&nbsp;&nbsp;在安装前，我们可以先敲下命令 git，看下看看系统有没有安装Git。<br />
安装方式一：<br />
&nbsp;&nbsp;&nbsp;&nbsp;如果你用Debian或Ubuntu Linux，通过一条sudo apt-get install git就可以直接完成Git的安装，非常简单。<br />
安装方式二：<br />
&nbsp;&nbsp;&nbsp;&nbsp;其他Linux版本，可以直接通过源码安装。<br />
先从Git官网下载源码，然后解压，依次输入以下这几个命令安装就好了：<br />
./config<br />
make<br />
sudo make install<br />

## Windows上安装Git
&nbsp;&nbsp;&nbsp;&nbsp;在Windows上使用Git，可以从Git官网直接[下载安装程序](https://git-scm.com/downloads)，（网速慢的同学请移步[国内镜像](https://pan.baidu.com/s/1kU5OCOB#list/path=%2Fpub%2Fgit)），然后按默认选项安装即可。<br />

安装完成后，在开始菜单里找到“Git”->“Git Bash”，蹦出一个类似命令行窗口的东西，就说明Git安装成功！<br />


