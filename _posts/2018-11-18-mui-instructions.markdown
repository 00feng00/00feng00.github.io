---
layout:       post
title:        "MUI使用教程"
subtitle:     "mui instructions"
date:         2018-11-18 10:05:00
author:       "ZeFeng"
header-img:   "img/ice_header_image.jpg"
header-mask:  0.3
catalog:      true
tags:
    - MUI
---

## 架构分析
技术栈架构图:<br/>
<img src="https://00feng00.github.io/img/mui-architecture.png">
项目架构分为三个部分：<br/>
1)客户端：在邮我行发布APP<br/>
2)客户端：发布IOS/Android的APP<br/>
3)服务端：JAVA数据交互层<br/>
具体分析：<br/>
一丶在邮我行发布APP<br/>
&nbsp;&nbsp;基于普元的壳子，我们使用webview，这样就可以使用Mui来开发我们的APP应用。（具体的代码后面会具体分析）<br/>
有了H5开发，所以，我们一套代码就可以在微信丶支付宝丶邮我行丶Android丶IOS上运行了。<br/>
H5的技术栈有好几种，这里个人推荐使用VUE，因为后面的更新迭代，我们可能会使用MUI最新的一套uni-app，它是基于Vue的，为了之后修改少部分的代码就可以更新迭代。之所以现在还不使用这套uni-app，因为它目前发布了小程序丶APP，还没有发布H5版本的，后面会发布的。<br/>
（额外的一点VUE3.0国内将在11.24发布，more faster more easier）<br/>
二丶使用MUI发布APP<br/>
&nbsp;&nbsp;我们使用H5+APP这套来开发，然后我们可以发布Android丶IOS的APP应用。<br/>
(MUI后面我们会讲很详细的)<br/>
三丶服务端<br/>
&nbsp;&nbsp;服务端我们使用JAVA，这里需要注意一点，后端开发人员要处理允许跨域调用，而且要允许option的访问，对option进行过滤。<br/>

## 体验
&nbsp;&nbsp;从上面的架构分析，我们可以很清晰的看到，我们是使用H5来开发的。<br/>
MUI UI 控件:<br/>
<img src="https://00feng00.github.io/img/mui-code-m.png">
UI 列表：<br/>
<img src="https://00feng00.github.io/img/mui-ui-01.png">
<img src="https://00feng00.github.io/img/mui-ui-02.png">
<img src="https://00feng00.github.io/img/mui-ui-03.png">
体验版：<br/>
可以体验我写出来的Demo,里面有一个调用生产的demo,调用用户登录查询接口。<br/>
后端开发人员需要注意一点：允许跨域调用，而且要允许option的访问，对option进行过滤。<br/>
