---
layout:       post
title:        "飞冰使用教程"
subtitle:     "ice instructions"
date:         2018-09-11 22:05:00
author:       "ZeFeng"
header-img:   "img/post-bg-os-metro.jpg"
header-mask:  0.3
catalog:      true
multilingual: true
tags:
    - 管理后台
    - VUE
    - ICE
---

## 前言
ICE是Alibaba 淘宝内部的一个开元脚手架，它的初始化脚手架我们称为模板，它提供了很多可复用代码片段(区块)，根据区块进行代码复用，可以大大节省开发时间，提高开发效率。
ICE提供了以下这些功能：
  1、模板自定义创建  2、区块可视化组装 3、 布局自定义生成  4、物料自定义接入 5、 项目仪表盘插件化
  <p style="color: blue;">个人推荐（尽量使用官方提供的模板和物料）</p>
## D2 Admin
  D2 Admin是一套管理系统脚手架，ICE推荐的Vue物料当中比较好的一个模板。这个是我们这篇文章讲的重点。
 
## 正文
下面开始讲如何使用ice 提供的D2 Admin。
## 安装
安装我们通过 ICE 推出的Iceworks 来快速安装我们的项目，目前支持 macOS 和 Windows 两大平台。下载链接[Iceworks](https://alibaba.github.io/ice/iceworks)
下载完成后，看下面两个图，进行想对应的操作：
<img src="https://00feng00.github.io/img/iceworker-use-saying-1.jpg">
<img src="https://00feng00.github.io/img/iceworker-use-saying-2.jpg">
## 运行
安装完成后，在Iceworks里面运行项目，就可以看到类似下面这样的界面：（只是类似，因为还需要修改的）
<img src="https://00feng00.github.io/img/ice_admin_bg.jpg">
## 注意
如果是前端进行开发，个人建议把依赖删掉，然后自己进行安装，步骤如下：
1、删除node_modules这个文件夹(最好使用命令行删除，不然有可能安装失败，有可能)
2、执行命令npm install
这样做的好处，自己对整个项目的结构很清晰，然后就可以按照自己的想法来操作了。
如果是后端进行开发，个人建议使用Iceworks这个GUI 软件来进行操作。因为一旦遇到问题，解决起来有那么一点点难度（虽然也不难，但是影响开发效率）。
## 开发
当你看到项目已经可以跑起来了，肯定已经有一定的信心来进行开发了，下面我们讲如何开发。
首先第一步，把官方提供的模板的骨架进行小小的修改：
1、修改浏览器标题、图标
根目录找到public文件夹，里面有两个html文件，第一个是加载中页面，一个是兼容移动端页面，我们在第一个imdex文件进行修改。
修改这4个地方：
```
    <link rel="icon" href="<%= BASE_URL %>logo.png">
    <title>数据中心管理后台</title>
    <div class="d2-app-loading-sub-title">欢迎使用 数据中心管理后台等在为您加载中。。。</div>
    <div class="d2-app-loading-sub-info">如果很久很久都没有加载成功，请清空缓存重新加载页面</div>
```
一个是修改图标（新增图标在public文件夹里面添加相对应的图片），一个是修改标题，下面这两个是修改文案提示。
_____________________________________________________
下面开始进入项目开发了






