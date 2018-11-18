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
<img src="https://00feng00.github.io/img/mui-architecture.png"><br/>
项目架构分为三个部分：<br/>
1)客户端：在邮我行发布APP<br/>
2)客户端：发布IOS/Android的APP<br/>
3)服务端：JAVA数据交互层<br/>
<br/>
具体分析：<br/>
一丶在邮我行发布APP<br/>
&nbsp;&nbsp;基于普元的壳子，我们使用webview，这样就可以使用Mui来开发我们的APP应用。（具体的代码后面会具体分析）<br/>
有了H5开发，所以，我们一套代码就可以在微信丶支付宝丶邮我行丶Android丶IOS上运行了。<br/>
H5的技术栈有好几种，这里个人推荐使用VUE，因为后面的更新迭代，我们可能会使用MUI最新的一套uni-app，它是基于Vue的，为了之后修改少部分的代码就可以更新迭代。之所以现在还不使用这套uni-app，因为它目前发布了小程序丶APP，还没有发布H5版本的，后面会发布的。<br/>
（额外的一点VUE3.0国内将在11.24发布，more faster more easier）<br/>
<br/>
二丶使用MUI发布APP<br/>
&nbsp;&nbsp;我们使用H5+APP这套来开发，然后我们可以发布Android丶IOS的APP应用。<br/>
(MUI后面我们会讲很详细的)<br/>
<br/>
三丶服务端<br/>
&nbsp;&nbsp;服务端我们使用JAVA，这里需要注意一点，后端开发人员要处理允许跨域调用，而且要允许option的访问，对option进行过滤。<br/>
<br/>
## 体验
&nbsp;&nbsp;从上面的架构分析，我们可以很清晰的看到，我们是使用H5来开发的。<br/>
MUI UI 控件:<br/>
<img src="https://00feng00.github.io/img/mui-code-m.png"> <br/>
<br/>
UI 列表：<br/>
<img src="https://00feng00.github.io/img/mui-ui-01.png">
<img src="https://00feng00.github.io/img/mui-ui-02.png">
<img src="https://00feng00.github.io/img/mui-ui-03.png">
体验版：<br/>
可以体验我写出来的Demo,里面有一个调用生产的demo,调用用户登录查询接口。<br/>
后端开发人员需要注意一点：允许跨域调用，而且要允许option的访问，对option进行过滤。<br/>

## 注意事项
在讲解代码之前，我们要注意几个点：<br/>
1丶固定栏靠前
&nbsp;&nbsp;&nbsp;&nbsp;固定栏，就是带有.mui-bar属性的节点，都是基于fixed定位的元素；<br/>
常见组件包括：<br/>
顶部导航栏（.mui-bar-nav）、底部工具条(.mui-bar-footer)、底部选项卡（.mui-bar-tab）;<br/>
这些元素使用时需遵循一个规则：<br/>
放在.mui-content元素之前，即使是底部工具条和底部选项卡，也要放在.mui-content之前，否则固定栏会遮住部分主内容；<br/>
2丶一切内容都要包裹在mui-content中<br/>
&nbsp;&nbsp;&nbsp;&nbsp;除了固定栏之外，其它内容都要包裹在.mui-content中，否则就有可能被固定栏遮罩，原因：固定栏基于Fixed定位，不受流式布局限制，普通内容依然会从top:0的位置开始布局，这样就会被固定栏遮罩，mui为了解决这个问题，定义了如下css代码：
```
    .mui-bar-nav ~ .mui-content {
        padding-top: 44px;
    }
    .mui-bar-footer ~ .mui-content {
        padding-bottom: 44px;
    }
    .mui-bar-tab ~ .mui-content {
        padding-bottom: 50px;
    }
```
当然拉，这个是H5开发，所以也可以自定义样式。everything is ok。<br/>

3丶始终为button按钮添加type属性<br/>
&nbsp;&nbsp;&nbsp;&nbsp;如果button按钮没有type属性，浏览器默认按照type=submit逻辑处理，这样若将没有type的button放在form表单中，点击按钮就会执行form表单提交，页面就会刷新，用户体验极差。

4丶页面初始化：必须执行mui.init方法<br/>
&nbsp;&nbsp;&nbsp;&nbsp;mui在页面初始化时，初始化了很多参数配置，比如：按键监听、手势监听等，因此mui页面都必须调用一次mui.init()方法；<br/>

5丶页面跳转：抛弃href跳转<br/>
&nbsp;&nbsp;&nbsp;&nbsp;建议使用mui.openWindow方法打开一个新的webview，mui会自动监听新页面的loaded事件，若加载完毕，再自动显示新页面；
有兴趣深入了解，拓展链接：<br/>
[hello mui中的无等待窗体切换是如何实现的](http://ask.dcloud.net.cn/article/106) <br/>
[提示HTML5的性能体验系列之一 避免切页白屏](http://ask.dcloud.net.cn/article/25) <br/>

6丶点击：忘记click<br/>
手机浏览器的click点击存在300毫秒延迟，mui为了解决这个问题，封装了tap事件，因此在任何点击的时候，请忘记click及onclick操作，统统使用如下代码：
```
    element.addEventListener('tap',function(){
        //点击响应逻辑
    });
```
这里讲解下，为什么click会有300ms：<br/>
双击缩放(double tap to zoom)<br/>
当用户一次点击屏幕之后，浏览器并不能立刻判断用户是要进行双击缩放，还是想要进行单击操作。因此，iOS Safari 就等待 300 毫秒，以判断用户是否再次点击了屏幕。<br/>
于是，300 毫秒延迟就这么诞生了。<br/>

最后有个点需要注意的：<br/>
mui为简化开发，将plusReady事件封装成了mui.plusReady()方法，凡涉及到HTML5+的api，建议都写在mui.plusReady方法中；<br/>
否则可能会报“plus is not defined”的错误；<br/>


## 正文
&nbsp;&nbsp;&nbsp;&nbsp;上面讲解了技术框架丶注意事项。下面我们开始进行详细的讲解。

## 使用webview
&nbsp;&nbsp;&nbsp;&nbsp;在邮我行发布项目，我们要基于H5来开发，所以使用webview。<br/>
Demo:<br/>

```
    <div id="container" width="100%" height="100%" layout="VBox" hAlign="center" vAlign="middle" >
        <webview id="webview" width="100%" height="100%" />
    </div>
    $M.page.addEvent('onLoad', function(params){
	// 设置网络URL
	webview.setUrl("http://172.20.10.6:8848/test/login.html?111");
    });
    // 设置状态栏颜色
    Utils.setStatusBarStyle('default');
```
代码分析：<br/>
我们使用了webview组件，通过设置url,就可以访问我们的应用。<br/>
可以自行通过setStatusBarStyle设置状态栏的颜色,有两种模式default/light<br/>

## 答疑
&nbsp;&nbsp;&nbsp;&nbsp;看到上面的代码，可能有的同事可以会问，那在邮我行发布，我们使用H5开发，怎么调用手机里面的Api呢。下面举个例子：<br/>
html:<br/>
```
    <div class="mui-page-content">
	<p id="scaleText">扫一扫后结果内容显示</p>
	<button id="onScale" onclick="getScaleData()" type="button" class="mui-btn mui-btn-green">扫一扫</button>
    </div>
```
js:<br/>
```
    function getScaleData () {
	// 调用邮我行的提供的API扫一扫
	Emp.execute("Utils.startBarCodeScanner(function(val){webview.execute('setPhTML(\"'+val+'\")')})");
    }
    function setPhTML (val) {
      scaleText.innerHTML = val;
    }
```
代码分析：<br/>
在页面写了个按钮和一个显示扫一扫后的结果显示文本。<br/>
通过按钮调用邮我行提供的扫一扫API，然后在扫完后调用H5页面的方法，把值设置到H5页面用来显示文本的地方就可以了。<br/>
<br/>
温馨提示：<br/>
上面的例子，我使用了onclick方法，调用事件，这里是给个反面的例子，尽量不要使用click,上面的注意事项也已经讲了。<br/>
所以我们的代码要这样写：<br/>
```
document.getElementById('onScale').addEventListener('tap', function() {
    Emp.execute("Utils.startBarCodeScanner(function(val){webview.execute('setPhTML(\"'+val+'\")')})");
})
```
<br/>
通过这种方式，如果在邮我行上发布的项目，要调用手机的API，我们就需要使用这种方式来实现。<br/>
如果是通过mui打包发行的我们直接使用mui提供的API就可以了，不需要调用邮我行的API。<br/>

## Mui API Reference
&nbsp;&nbsp;&nbsp;&nbsp;通过扫描上面的UI二维码，我们可以看到MUI的UI控件是很齐全的，基本市面上有的，它基本都有。<br/>
UI控件的使用，这里就不额外讲解了，把文档的Demo拉下来就能看到效果了。<br/>
Mui提供了以下调用手机API的接口，看图：<br/>
<br/>
<img src="https://00feng00.github.io/img/mui-interface-01.png">
<img src="https://00feng00.github.io/img/mui-interface-02.png">
<img src="https://00feng00.github.io/img/mui-interface-03.png">
<img src="https://00feng00.github.io/img/mui-interface-04.png">
<br/>
<br/>
上面的接口参考，我们可以把例子拉下来就可以运行的拉，但是记得一点，这些接口要使用mui打包才能使用。<br/>
如果是在邮我行上发布的，我们还是得使用上面的例子。<br/>

## Mui Ajax


## 结语
文末，个人建议：减少css二次渲染，就是少用复杂的选择器，少用padding、margin这些会二次修正页面的css。
如果追求极致的话，那jquery、zepto这些框架也不要使用，手机上都是webkit引擎，直接写document的api操作dom即没有兼容问题又没有效率问题。









