---
layout:       post
title:        "Vue响应式原理"
subtitle:     "vue responsive principle"
date:         2018-10-18 22:05:00
author:       "ZeFeng"
header-img:   "img/ice_header_image.jpg"
header-mask:  0.3
catalog:      true
tags:
    - VUE响应式原理
---

## 前言
Vue最独特的，非侵入性的响应式系统莫属了。数据模型仅仅是普通的 JavaScript 对象，修改它们时，视图会进行更新，这样就使得状态管理非常简单直接。

## 正文

<b>追踪变化</b>
当我们把一个普通的 JavaScript 对象传给 Vue 实例的 data 选项的时候，Vue 将遍历此对象所有的属性，并使用 Object.defineProperty 把这些属性全部转为 getter/setter。
写法：
```
var vm = new Vue({
  data:{
    a:1
  }
})
```
| 方法 | 说明 | 备注 |
|-----|-----|------|
| Object.defineProperty | 允许精确添加或修改对象的属性  | nice |
| getter/setter | 对象的属性是由名字、值和一组特性（可写、可枚举、可配置等）构成的，属性值可以用一个或两个方法代替，这两个方法就是getter和setter   | 默认情况下，使用 Object.defineProperty() 添加的属性值是不可修改的 |


