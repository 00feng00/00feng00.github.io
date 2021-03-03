---
layout:       post
title:        "Vue 3.0 指令源码解析"
subtitle:     "vue3 directive"
date:         2018-10-14 22:05:00
author:       "ZeFeng"
header-img:   "img/websocket_logo.jpg"
header-mask:  0.3
catalog:      true
tags:
    - VUE3
    - 源码
---
在 Vue 的项目中，我们经常会遇到 `v-if`、`v-show`、`v-for` 或 `v-model` 这些内置指令，它们为我们提供了不同的功能。除了使用这些内置指令之外，Vue 也允许注册自定义指令。

接下来，使用 Vue 3 官方文档 **自定义指令** 章节中使用的示例，来一步步揭开自定义指令背后的秘密。

> 提示：在阅读本文前，建议您先阅读 Vue 3 官方文档 **自定义指令** 章节的内容。

### 一、自定义指令

**1、注册全局自定义指令**

```
`const app = Vue.createApp({})  
  
// 注册一个全局自定义指令 v-focus  
app.directive('focus', {  
  // 当被绑定的元素挂载到 DOM 中时被调用  
  mounted(el) {  
    // 聚焦元素  
    el.focus()  
  }  
})  
`
```

**2、使用全局自定义指令**

```
`<div id="app">  
   <input v-focus />  
</div>  
`
```

**3、完整的使用示例**

```
`<div id="app">  
   <input v-focus />  
</div>  
<script> const { createApp } = Vue  
     
   const app = Vue.createApp({}) // ①  
   app.directive('focus', { // ②   
     // 当被绑定的元素挂载到 DOM 中时被调用  
     mounted(el) {  
       el.focus() // 聚焦元素  
     }  
   })  
   app.mount('#app') // ③</script>  
`
```

当页面加载完成后，页面中的输入框元素将自动获得焦点。该示例的代码比较简单，主要包含 3 个步骤：**创建 App 对象、注册全局自定义指令和应用挂载**。其中创建 App 对象的细节，在后续的文章中单独介绍，下面我们将重点分析其他 2 个步骤。首先我们先来分析注册全局自定义指令的过程。

### 二、注册全局自定义指令的过程

在以上示例中，我们使用 `app` 对象的 `directive` 方法来注册全局自定义指令：

```
`app.directive('focus', {  
  // 当被绑定的元素挂载到 DOM 中时被调用  
  mounted(el) {  
    el.focus() // 聚焦元素  
  }  
})  
`
```

当然，除了注册全局自定义指令外，我们也可以注册局部指令，因为组件中也接受一个 `directives` 的选项：

```
`directives: {  
  focus: {  
    mounted(el) {  
      el.focus()  
    }  
  }  
}  
`
```

对于以上示例来说，我们使用的 `app.directive` 方法被定义在 `runtime-core/src/apiCreateApp.ts` 文件中：

```
``// packages/runtime-core/src/apiCreateApp.ts  
export function createAppAPI<HostElement>( render: RootRenderFunction,  
  hydrate?: RootHydrateFunction): CreateAppFunction<HostElement> {  
  return function createApp(rootComponent, rootProps = null) {  
    const context = createAppContext()  
    let isMounted = false  
  
    const app: App = (context.app = {  
      // 省略部分代码  
      _context: context,  
        
      // 用于注册或检索全局指令。  
      directive(name: string, directive?: Directive) {  
        if (__DEV__) {  
          validateDirectiveName(name)  
        }  
        if (!directive) {  
          return context.directives[name] as any  
        }  
        if (__DEV__ && context.directives[name]) {  
          warn(`Directive "${name}" has already been registered in target app.`)  
        }  
        context.directives[name] = directive  
        return app  
      },  
  
    return app  
  }  
}  
``
```

通过观察以上代码，我们可以知道 `directive` 方法支持以下两个参数：

*   name：表示指令的名称；
    
*   directive（可选）：表示指令的定义。
    

name 参数比较简单，所以我们重点分析 `directive` 参数，该参数的类型是 `Directive` 类型：

```
`// packages/runtime-core/src/directives.ts  
export type Directive<T = any, V = any> =  
  | ObjectDirective<T, V>  
  | FunctionDirective<T, V>  
`
```

由上可知 `Directive` 类型属于联合类型，所以我们需要继续分析 `ObjectDirective` 和 `FunctionDirective` 类型。这里我们先来看一下 `ObjectDirective` 类型的定义：

```
`// packages/runtime-core/src/directives.ts  
export interface ObjectDirective<T = any, V = any> {  
  created?: DirectiveHook<T, null, V>  
  beforeMount?: DirectiveHook<T, null, V>  
  mounted?: DirectiveHook<T, null, V>  
  beforeUpdate?: DirectiveHook<T, VNode<any, T>, V>  
  updated?: DirectiveHook<T, VNode<any, T>, V>  
  beforeUnmount?: DirectiveHook<T, null, V>  
  unmounted?: DirectiveHook<T, null, V>  
  getSSRProps?: SSRDirectiveHook  
}  
`
```

该类型定义了对象类型的指令，对象上的每个属性表示指令生命周期上的钩子。而 `FunctionDirective` 类型则表示函数类型的指令：

```
`// packages/runtime-core/src/directives.ts  
export type FunctionDirective<T = any, V = any> = DirectiveHook<T, any, V>  
                                
export type DirectiveHook<T = any, Prev = VNode<any, T> | null, V = any> = (  
  el: T,  
  binding: DirectiveBinding<V>,  
  vnode: VNode<any, T>,  
  prevVNode: Prev  
) => void` 
```

介绍完 `Directive` 类型，我们再回顾一下前面的示例，相信你就会清晰很多：

```
`app.directive('focus', {  
  // 当被绑定的元素挂载到 DOM 中时触发  
  mounted(el) {  
    el.focus() // 聚焦元素  
  }  
})  
`
```

对于以上示例，当我们调用 `app.directive` 方法注册自定义 `focus` 指令时，就会执行以下逻辑：

```
``directive(name: string, directive?: Directive) {  
  if (__DEV__) { // 避免自定义指令名称，与已有的内置指令名称冲突  
    validateDirectiveName(name)  
  }  
  if (!directive) { // 获取name对应的指令对象  
    return context.directives[name] as any  
  }  
  if (__DEV__ && context.directives[name]) {  
    warn(`Directive "${name}" has already been registered in target app.`)  
  }  
  context.directives[name] = directive // 注册全局指令  
  return app  
}  
``
```

当 `focus` 指令注册成功之后，该指令会被保存在 `context` 对象的 `directives` 属性中，具体如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V37dTncbsSHhsibNbic3bVCRibTc0lZPu2yvxjzcLjI4mdU3Siabq4Y9o5wWwhfQzro6Eq3bsFqFuLROw/640?wx_fmt=jpeg)

顾名思义 `context` 是表示应用的上下文对象，那么该对象是如何创建的呢？其实，该对象是通过 `createAppContext` 函数来创建的：

```
`const context = createAppContext()  
`
```

而 `createAppContext` 函数被定义在 `runtime-core/src/apiCreateApp.ts` 文件中：

```
`// packages/runtime-core/src/apiCreateApp.ts  
export function createAppContext(): AppContext {  
  return {  
    app: null as any,  
    config: {  
      isNativeTag: NO,  
      performance: false,  
      globalProperties: {},  
      optionMergeStrategies: {},  
      isCustomElement: NO,  
      errorHandler: undefined,  
      warnHandler: undefined  
    },  
    mixins: [],  
    components: {},  
    directives: {},  
    provides: Object.create(null)  
  }  
}  
`
```

看到这里，是不是觉得注册全局自定义指令的内部处理逻辑其实挺简单的。那么对于已注册的 `focus` 指令，何时会被调用呢？要回答这个问题，我们就需要分析另一个步骤 —— **应用挂载**。

### 三、应用挂载的过程

为了更加直观地了解应用挂载的过程，利用 **Chrome 开发者工具**，记录了应用挂载的主要过程：

![](https://mmbiz.qpic.cn/mmbiz_png/jQmwTIFl1V37dTncbsSHhsibNbic3bVCRibhKh2o5l3Vtic5877cW10IHxibDTJwTScuIhOnlExI16d6bD5tc3uTUNg/640?wx_fmt=png)

通过上图，我们就可以知道应用挂载期间所经历的主要过程。此外，从图中我们也发现了一个与指令相关的函数 `resolveDirective`。很明显，该函数用于解析指令，且该函数在 `render` 方法中会被调用。在源码中，我们找到了该函数的定义：

```
`// packages/runtime-core/src/helpers/resolveAssets.ts  
export function resolveDirective(name: string): Directive | undefined {  
  return resolveAsset(DIRECTIVES, name)  
}  
`
```

在 `resolveDirective` 函数内部，会继续调用 `resolveAsset` 函数来执行具体的解析操作。在分析 `resolveAsset` 函数的具体实现之前，我们在 `resolveDirective` 函数内部加个断点，来一睹 `render` 方法的 “芳容”：

![](https://mmbiz.qpic.cn/mmbiz_png/jQmwTIFl1V37dTncbsSHhsibNbic3bVCRibJT3bTZkfGoChVd47ukrjWVaIaIic5vVFKsE3icicf9ekR0Wzo80rF9cbw/640?wx_fmt=png)

在上图中，我们看到了与 `focus` 指令相关的 `_resolveDirective("focus")` 函数调用。前面我们已经知道在 `resolveDirective` 函数内部会继续调用 `resolveAsset` 函数，该函数的具体实现如下：

```
``// packages/runtime-core/src/helpers/resolveAssets.ts  
function resolveAsset( type: typeof COMPONENTS | typeof DIRECTIVES,  
  name: string,  
  warnMissing = true) {  
  const instance = currentRenderingInstance || currentInstance  
  if (instance) {  
    const Component = instance.type  
    // 省略解析组件的处理逻辑  
    const res =  
      // 局部注册  
      resolve(instance[type] || (Component as ComponentOptions)[type], name) ||  
      // 全局注册  
      resolve(instance.appContext[type], name)  
    return res  
  } else if (__DEV__) {  
    warn(  
      `resolve${capitalize(type.slice(0, -1))} ` +  
        `can only be used in render() or setup().`  
    )  
  }  
}  
``
```

因为注册 `focus` 指令时，使用的是全局注册的方式，所以解析的过程会执行 `resolve(instance.appContext[type], name)` 该语句，其中 `resolve` 方法的定义如下：

```
`function resolve(registry: Record<string, any> | undefined, name: string) {  
  return (  
    registry &&  
    (registry[name] ||  
      registry[camelize(name)] ||  
      registry[capitalize(camelize(name))])  
  )  
}  
`
```

分析完以上的处理流程，我们可以知道在解析全局注册的指令时，会通过 `resolve` 函数从应用的上下文对象中获取已注册的指令对象。在获取到 `_directive_focus` 指令对象后，`render` 方法内部会继续调用 `_withDirectives` 函数，用于把指令添加到 `VNode` 对象上，该函数被定义在 `runtime-core/src/directives.ts` 文件中：

```
`// packages/runtime-core/src/directives.ts  
export function withDirectives<T extends VNode>( vnode: T,  
  directives: DirectiveArguments): T {  
  const internalInstance = currentRenderingInstance // 获取当前渲染的实例  
  const instance = internalInstance.proxy  
  const bindings: DirectiveBinding[] = vnode.dirs || (vnode.dirs = [])  
  for (let i = 0; i < directives.length; i++) {  
    let [dir, value, arg, modifiers = EMPTY_OBJ] = directives[i]  
    // 在 mounted 和 updated 时，触发相同行为，而不关系其他的钩子函数  
    if (isFunction(dir)) { // 处理函数类型指令  
      dir = {  
        mounted: dir,  
        updated: dir  
      } as ObjectDirective  
    }  
    bindings.push({  
      dir,  
      instance,  
      value,  
      oldValue: void 0,  
      arg,  
      modifiers  
    })  
  }  
  return vnode  
}  
`
```

因为一个节点上可能会应用多个指令，所以 `withDirectives` 函数在 `VNode` 对象上定义了一个 `dirs` 属性且该属性值为数组。对于前面的示例来说，在调用 `withDirectives` 函数之后，`VNode` 对象上就会新增一个 `dirs` 属性，具体如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V37dTncbsSHhsibNbic3bVCRibCpk27ZuicZNOHaowEdrZicaf8ExiaYfH0mmJfXeZsY14HfBhsHyBjGTZw/640?wx_fmt=jpeg)

通过上面的分析，我们已经知道在组件的 `render` 方法中，我们会通过  `withDirectives` 函数把指令注册对应的 `VNode` 对象上。那么 `focus` 指令上定义的钩子什么时候会被调用呢？在继续分析之前，我们先来介绍一下指令对象所支持的钩子函数。

一个指令定义对象可以提供如下几个钩子函数 (均为可选)：

*   `created`：在绑定元素的属性或事件监听器被应用之前调用。
    
*   `beforeMount`：当指令第一次绑定到元素并且在挂载父组件之前调用。
    
*   `mounted`：在绑定元素的父组件被挂载后调用。
    
*   `beforeUpdate`：在更新包含组件的 VNode 之前调用。
    
*   `updated`：在包含组件的 VNode 及其子组件的 VNode 更新后调用。
    
*   `beforeUnmount`：在卸载绑定元素的父组件之前调用。
    
*   `unmounted`：当指令与元素解除绑定且父组件已卸载时，只调用一次。
    

介绍完这些钩子函数之后，我们再来回顾一下前面介绍的 `ObjectDirective` 类型：

```
`// packages/runtime-core/src/directives.ts  
export interface ObjectDirective<T = any, V = any> {  
  created?: DirectiveHook<T, null, V>  
  beforeMount?: DirectiveHook<T, null, V>  
  mounted?: DirectiveHook<T, null, V>  
  beforeUpdate?: DirectiveHook<T, VNode<any, T>, V>  
  updated?: DirectiveHook<T, VNode<any, T>, V>  
  beforeUnmount?: DirectiveHook<T, null, V>  
  unmounted?: DirectiveHook<T, null, V>  
  getSSRProps?: SSRDirectiveHook  
}  
`
```

好的，接下来我们来分析一下 `focus` 指令上定义的钩子什么时候被调用。同样，在 `focus` 指令的 `mounted` 方法中加个断点：

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V37dTncbsSHhsibNbic3bVCRibrvKbWI6fC57oVL2ew3OmD2CNDJW1sJoRJYbrMeZBZf6YoK3LbEHiaEQ/640?wx_fmt=jpeg)

在图中右侧的调用栈中，我们看到了 `invokeDirectiveHook` 函数，很明显该函数的作用就是调用指令上已注册的钩子。出于篇幅考虑，具体的细节就不继续介绍了，感兴趣的小伙伴可以自行断点调试一下。

### 四、有话说

#### 4.1 Vue 3 有哪些内置指令？

在介绍注册全局自定义指令的过程中，我们看到了一个 `validateDirectiveName` 函数，该函数用于验证自定义指令的名称，从而避免自定义指令名称，与已有的内置指令名称冲突。

```
`// packages/runtime-core/src/directives.ts  
export function validateDirectiveName(name: string) {  
  if (isBuiltInDirective(name)) {  
    warn('Do not use built-in directive ids as custom directive id: ' + name)  
  }  
}  
`
```

在 `validateDirectiveName` 函数内部，会通过 `isBuiltInDirective(name)` 语句来判断是否为内置指令：

```
`const isBuiltInDirective = /*#__PURE__*/ makeMap(  
  'bind,cloak,else-if,else,for,html,if,model,on,once,pre,show,slot,text'  
)  
`
```

以上代码中的 `makeMap` 函数，用于生成一个 map 对象（Object.create(null)）并返回一个函数，用于检测某个 key 是否存在 map 对象中。另外，通过以上代码，我们就可以很清楚地了解 Vue 3 中为我们提供了哪些内置指令。

#### 4.2 指令有几种类型？

在 Vue 3 中指令分为 `ObjectDirective` 和 `FunctionDirective` 两种类型：

```
`// packages/runtime-core/src/directives.ts  
export type Directive<T = any, V = any> =  
  | ObjectDirective<T, V>  
  | FunctionDirective<T, V>  
`
```

##### ObjectDirective

```
`export interface ObjectDirective<T = any, V = any> {  
  created?: DirectiveHook<T, null, V>  
  beforeMount?: DirectiveHook<T, null, V>  
  mounted?: DirectiveHook<T, null, V>  
  beforeUpdate?: DirectiveHook<T, VNode<any, T>, V>  
  updated?: DirectiveHook<T, VNode<any, T>, V>  
  beforeUnmount?: DirectiveHook<T, null, V>  
  unmounted?: DirectiveHook<T, null, V>  
  getSSRProps?: SSRDirectiveHook  
}  
`
```

##### FunctionDirective

```
`export type FunctionDirective<T = any, V = any> = DirectiveHook<T, any, V>  
                                
export type DirectiveHook<T = any, Prev = VNode<any, T> | null, V = any> = (  
  el: T,  
  binding: DirectiveBinding<V>,  
  vnode: VNode<any, T>,  
  prevVNode: Prev  
) => void  
`
```

如果你想在 `mounted` 和 `updated` 时触发相同行为，而不关心其他的钩子函数。那么你可以通过将回调函数传递给指令来实现：

```
`app.directive('pin', (el, binding) => {  
  el.style.position = 'fixed'  
  const s = binding.arg || 'top'  
  el.style[s] = binding.value + 'px'  
})  
`
```

#### 4.3 注册全局指令与局部指令有什么区别？

##### 注册全局指令

```
`app.directive('focus', {  
  // 当被绑定的元素挂载到 DOM 中时被调用  
  mounted(el) {  
    el.focus() // 聚焦元素  
  }  
});  
`
```

##### 注册局部指令

```
`const Component = defineComponent({  
  directives: {  
    focus: {  
      mounted(el) {  
        el.focus()  
      }  
    }  
  },  
  render() {  
    const { directives } = this.$options;  
    return [withDirectives(h('input'), [[directives.focus, ]])]  
  }  
});  
`
```

##### 解析全局注册和局部注册的指令

```
`// packages/runtime-core/src/helpers/resolveAssets.ts  
function resolveAsset( type: typeof COMPONENTS | typeof DIRECTIVES,  
  name: string,  
  warnMissing = true) {  
  const instance = currentRenderingInstance || currentInstance  
  if (instance) {  
    const Component = instance.type  
    // 省略解析组件的处理逻辑  
    const res =  
      // 局部注册  
      resolve(instance[type] || (Component as ComponentOptions)[type], name) ||  
      // 全局注册  
      resolve(instance.appContext[type], name)  
    return res  
  }  
}  
`
```

#### 4.4 内置指令和自定义指令生成的渲染函数有什么区别？

要了解内置指令和自定义指令生成的渲染函数的区别，以 `v-if` 、`v-show` 内置指令和 `v-focus` 自定义指令为例，然后使用 Vue 3 Template Explorer 这个在线工具来编译生成渲染函数：

##### v-if 内置指令

```
`<input v-if="isShow" />  
  
const _Vue = Vue  
return function render(_ctx, _cache, $props, $setup, $data, $options) {  
  with (_ctx) {  
    const { createVNode: _createVNode, openBlock: _openBlock,   
      createBlock: _createBlock, createCommentVNode: _createCommentVNode } = _Vue  
  
    return isShow  
      ? (_openBlock(), _createBlock("input", { key: 0 }))  
      : _createCommentVNode("v-if", true)  
  }  
}  
`
```

对于 `v-if` 指令来说，在编译后会通过 `?:` 三目运算符来实现动态创建节点的功能。

##### v-show 内置指令

```
`<input v-show="isShow" />  
    
const _Vue = Vue  
return function render(_ctx, _cache, $props, $setup, $data, $options) {  
  with (_ctx) {  
    const { vShow: _vShow, createVNode: _createVNode, withDirectives: _withDirectives,   
      openBlock: _openBlock, createBlock: _createBlock } = _Vue  
  
    return _withDirectives((_openBlock(), _createBlock("input", null, null, 512 /* NEED_PATCH */)), [  
      [_vShow, isShow]  
    ])  
  }  
}  
`
```

以上示例中的 `vShow` 指令被定义在 `packages/runtime-dom/src/directives/vShow.ts` 文件中，该指令属于 `ObjectDirective` 类型的指令，该指令内部定义了 `beforeMount`、`mounted`、`updated` 和 `beforeUnmount` 四个钩子。

##### v-focus 自定义指令

```
`<input v-focus />  
  
const _Vue = Vue  
return function render(_ctx, _cache, $props, $setup, $data, $options) {  
  with (_ctx) {  
    const { resolveDirective: _resolveDirective, createVNode: _createVNode,   
      withDirectives: _withDirectives, openBlock: _openBlock, createBlock: _createBlock } = _Vue  
  
    const _directive_focus = _resolveDirective("focus")  
    return _withDirectives((_openBlock(), _createBlock("input", null, null, 512 /* NEED_PATCH */)), [  
      [_directive_focus]  
    ])  
  }  
}  
`
```

通过对比 `v-focus` 与 `v-show` 指令生成的渲染函数，我们可知 `v-focus` 自定义指令与 `v-show` 内置指令都会通过 `withDirectives` 函数，把指令注册到 `VNode` 对象上。而自定义指令相比内置指令来说，会多一个指令解析的过程。

此外，如果在 `input` 元素上，同时应用了 `v-show` 和 `v-focus` 指令，则在调用 `_withDirectives` 函数时，将使用二维数组：

```
`<input v-show="isShow" v-focus />  
  
const _Vue = Vue  
return function render(_ctx, _cache, $props, $setup, $data, $options) {  
  with (_ctx) {  
    const { vShow: _vShow, resolveDirective: _resolveDirective, createVNode: _createVNode,   
      withDirectives: _withDirectives, openBlock: _openBlock, createBlock: _createBlock } = _Vue  
  
    const _directive_focus = _resolveDirective("focus")  
    return _withDirectives((_openBlock(), _createBlock("input", null, null, 512 /* NEED_PATCH */)), [  
      [_vShow, isShow],  
      [_directive_focus]  
    ])  
  }  
}  
`
```

#### 4.5 如何在渲染函数中应用指令？

除了在模板中应用指令之外，利用前面介绍的 `withDirectives` 函数，我们可以很方便地在渲染函数中应用指定的指令：

```
`<div id="app"></div>  
<script> const { createApp, h, vShow, defineComponent, withDirectives } = Vue  
   const Component = defineComponent({  
     data() {  
       return { value: true }  
     },  
     render() {  
       return [withDirectives(h('div', '我是'), [[vShow, this.value]])]  
     }  
   });  
   const app = Vue.createApp(Component)  
   app.mount('#app')</script>  
`
```

本文主要介绍了在 Vue 3 中如何自定义指令、如何注册全局和局部指令。为了让大家能够更深入地掌握自定义指令的相关知识，从源码的角度分析了指令的注册和应用过程。

在后续的文章中，将会介绍一些特殊的指令，当然也会重点分析一下双向绑定的原理，感兴趣的小伙伴不要错过哟。

### 五、参考资源

*   Vue 3 官网 - 自定义指令
    
*   Vue 3 官网 - 应用 API
