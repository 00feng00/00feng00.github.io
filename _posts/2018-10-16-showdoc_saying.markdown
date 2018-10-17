---
layout:       post
title:        "showdoc saying"
subtitle:     "showdoc instructions"
date:         2018-10-16 22:05:00
author:       "ZeFeng"
header-img:   "img/ice_header_image.jpg"
header-mask:  0.3
catalog:      true
tags:
    - showdoc
    - teamwork
---


## 背景
&nbsp;&nbsp;我们团队的API文档基本都是用txt或者world,有一些是通过当面沟通定下的，这种方式其实不太友好。最近我在开发数据中心的时候，这种感觉更加深刻。
一个项目下来，那些txt或者world就会被丢弃或者丢失，这可能会产生几个问题：<br />
1、二次开发沟通成本大<br />
2、文档不能留底，定位问题花费时间多<br />
3、不利于团队协作<br />
txt或者world不是一个很好的选择。我有一个感觉，我们团队走的方向应该会趋向专业化。为了让我们的团队开发更加高效，同时减少我们的沟通成本，加快团队之间沟通的效率
个人觉得应该需要作出一些调整。

## 产品选型
&nbsp;&nbsp;我对比了几个产品，showdoc应该是比较适合我们团队的。基于以下几点考虑：
1、ShowDoc采用markdown编辑器，无论是编辑还是阅读体验都极佳。
我们团队的开发人员对markdown都挺熟悉的（markdown学习成本基本可以忽略不计，很容易掌握）。
2、ShowDoc的模板插入，让我们可以花很少的时间插入API接口模板和数据字典模板，然后就是改动数据。
3、ShowDoc的使用可以在线，也可以安装到自己的服务器，我们可以建立我们自己的ShowDoc。

## 好处
使用ShowDoc有几个好处：
1、后端开发的数据设计有提高，在开发之前要先定下开发的数据结构，对数据处理更加容易把握。
2、对测试人员有一个很大的帮助，对职业生涯都有一个很大的提升。这里详细讲下，有了这个，测试人员就可以进行接口测试了，进而自动化也就是下一步的目标了。
这里介绍三个测试的东东
1)、 [mocha](https://m.aliyun.com/jiaocheng/topic_75092_1.html)
2)、 [karma](http://karma-runner.github.io/2.0/index.html)
3)、 [ava](http://i5ting.github.io/ava-practice/)
3、对前端更加友好，你懂的
4、加快团队之间沟通的效率

## 展示
API文档
<img src="https://00feng00.github.io/img/showdoc_interface-saying.jpg">
数据字典
<img src="https://00feng00.github.io/img/showdoc_data-saying.jpg">

## ShowDoc使用手册









