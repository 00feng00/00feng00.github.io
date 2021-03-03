---
layout:       post
title:        "Vue3.0动态组件"
subtitle:     "Vue3 component"
date:         2021-03-02 22:05:00
author:       "ZeFeng"
header-img:   "img/websocket_logo.jpg"
header-mask:  0.3
catalog:      true
tags:
    - VUE3
    - 源码
---
本文是 **Vue 3.0 进阶系列** 的第四篇文章，在这篇文章中，将介绍 Vue 3 中的内置组件 —— `component`，该组件的作用是渲染一个 “元组件” 为动态组件。如果你对动态组件还不了解的话也没关系，文中会通过具体的示例，来介绍动态组件的应用。

由于动态组件内部与组件注册之间有一定的联系，所以为了让大家能够更好地了解动态组件的内部原理，会先介绍组件注册的相关知识。  

### 一、组件注册

#### 1.1 全局注册

在 Vue 3.0 中，通过使用 `app` 对象的 `component` 方法，可以很容易地注册或检索全局组件。`component` 方法支持两个参数：

*   name：组件名称；
    
*   component：组件定义对象。
    

接下来，我们来看一个简单的示例：

```
<div id="app">  
   <component-a></component-a>  
   <component-b></component-b>  
   <component-c></component-c>  
</div>  
<script> const { createApp } = Vue  
   const app = createApp({}); // ①  
   app.component('component-a', { // ②  
     template: "<p>我是组件A</p>"  
   });  
   app.component('component-b', {  
     template: "<p>我是组件B</p>"  
   });  
   app.component('component-c', {  
     template: "<p>我是组件C</p>"  
   });  
   app.mount('#app') // ③</script>  
```

在以上代码中，我们通过 `app.component` 方法注册了 3 个组件，这些组件都是全局注册的 。**也就是说它们在注册之后可以用在任何新创建的组件实例的模板中**。

该示例的代码比较简单，主要包含 3 个步骤：**创建 App 对象、注册全局组件和应用挂载**。其中创建 `App` 对象的细节，会在后续的文章中单独介绍，下面我们将重点分析其他 2 个步骤，首先我们先来分析注册全局组件的过程。

#### 1.2 注册全局组件的过程

在以上示例中，我们使用 `app` 对象的 `component` 方法来注册全局组件：

```
app.component('component-a', {  
  template: "<p>我是组件A</p>"  
});  

```

当然，除了注册全局组件之外，我们也可以注册局部组件，因为组件中也接受一个 `components` 的选项：

```
const app = Vue.createApp({  
  components: {  
    'component-a': ComponentA,  
    'component-b': ComponentB  
  }  
})  

```

**需要注意的是，局部注册的组件在其子组件中是不可用的**。接下来，我们来继续介绍注册全局组件的过程。对于前面的示例来说，我们使用的 `app.component` 方法被定义在 `runtime-core/src/apiCreateApp.ts` 文件中：

```
export function createAppAPI<HostElement>( render: RootRenderFunction,  
  hydrate?: RootHydrateFunction): CreateAppFunction<HostElement> {  
  return function createApp(rootComponent, rootProps = null) {  
    const context = createAppContext()  
    const installedPlugins = new Set()  
    let isMounted = false  
  
    const app: App = (context.app = {  
      // 省略部分代码  
      _context: context,  
  
      // 注册或检索全局组件  
      component(name: string, component?: Component): any {  
        if (__DEV__) {  
          validateComponentName(name, context.config)  
        }  
        if (!component) { // 获取name对应的组件  
          return context.components[name]  
        }  
        if (__DEV__ && context.components[name]) { // 重复注册提示  
          warn(`Component "${name}" has already been registered in target app.`)  
        }  
        context.components[name] = component // 注册全局组件  
        return app  
      },  
    })  
  
    return app  
  }  
}  

```

当所有的组件都注册成功之后，它们会被保存到 `context` 对象的 `components` 属性中，具体如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V3avmCJr5XatqAuVMKMqVTHN4TFmw5lnXp3dpaIIUlkVPs0nnq9rjqYGLl5kFCTtsJEvD0r0icSmHA/640?wx_fmt=jpeg)

顾名思义 `context` 是表示应用的上下文对象，那么该对象是如何创建的呢？其实，该对象是通过 `createAppContext` 函数来创建的：

```
const context = createAppContext()  

```

而 `createAppContext` 函数被定义在 `runtime-core/src/apiCreateApp.ts` 文件中：

```
// packages/runtime-core/src/apiCreateApp.ts  
export function createAppContext(): AppContext {  
  return {  
    app: null as any,  
    config: { // 应用的配置对象  
      isNativeTag: NO,  
      performance: false,  
      globalProperties: {},  
      optionMergeStrategies: {},  
      isCustomElement: NO,  
      errorHandler: undefined,  
      warnHandler: undefined  
    },  
    mixins: [], // 保存应用内的混入  
    components: {}, // 保存全局组件的信息  
    directives: {}, // 保存全局指令的信息  
    provides: Object.create(null)  
  }  
}  

```

分析完 `app.component` 方法之后，是不是觉得组件注册的过程还是挺简单的。那么对于已注册的组件，何时会被使用呢？要回答这个问题，我们就需要分析另一个步骤 —— **应用挂载**。

#### 1.3 应用挂载的过程

为了更加直观地了解应用挂载的过程，利用 Chrome 开发者工具的 **Performance** 标签栏，记录了应用挂载的主要过程：

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V3avmCJr5XatqAuVMKMqVTHiayKdrAP3kd7Kz2mvApMYn5ibefiaGozbEVaibJwCVhdQMVSHFquJtpPkw/640?wx_fmt=jpeg)

在上图中我们发现了一个与组件相关的函数 `resolveComponent`。很明显，该函数用于解析组件，且该函数在 `render` 方法中会被调用。在源码中，我们找到了该函数的定义：

```
// packages/runtime-core/src/helpers/resolveAssets.ts  
const COMPONENTS = 'components'  
  
export function resolveComponent(name: string): ConcreteComponent | string {  
  return resolveAsset(COMPONENTS, name) || name  
}  

```

由以上代码可知，在 `resolveComponent` 函数内部，会继续调用 `resolveAsset` 函数来执行具体的解析操作。在分析 `resolveAsset` 函数的具体实现之前，我们在 `resolveComponent` 函数内部加个断点，来一睹 `render` 方法的 “芳容”：

![](https://mmbiz.qpic.cn/mmbiz_png/jQmwTIFl1V3avmCJr5XatqAuVMKMqVTHGmAlicJrXMA8PenjL74rFed1o7M5LXJ8nARnTbFYUZIWUpfibpYBJbEA/640?wx_fmt=png)

在上图中，我们看到了解析组件的操作，比如 `_resolveComponent("component-a")`。前面我们已经知道在 `resolveComponent` 函数内部会继续调用 `resolveAsset` 函数，该函数的具体实现如下：

```
// packages/runtime-core/src/helpers/resolveAssets.ts  
function resolveAsset( type: typeof COMPONENTS | typeof DIRECTIVES,  
  name: string,  
  warnMissing = true) {  
  const instance = currentRenderingInstance || currentInstance  
  if (instance) {  
    const Component = instance.type  
    // 省略大部分处理逻辑  
    const res =  
      // 局部注册  
      // check instance[type] first for components with mixin or extends.  
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

```

因为注册组件时，使用的是全局注册的方式，所以解析的过程会执行 `resolve(instance.appContext[type], name)` 该语句，其中 `resolve` 方法的定义如下：

```
// packages/runtime-core/src/helpers/resolveAssets.ts  
function resolve(registry: Record<string, any> | undefined, name: string) {  
  return (  
    registry &&  
    (registry[name] ||  
      registry[camelize(name)] ||  
      registry[capitalize(camelize(name))])  
  )  
}  

```

分析完以上的处理流程，我们在解析全局注册的组件时，会通过 `resolve` 函数从应用的上下文对象中获取已注册的组件对象。

```
(function anonymous() {  
    const _Vue = Vue  
  
    return function render(_ctx, _cache) {  
        with (_ctx) {  
          const {resolveComponent: _resolveComponent, createVNode: _createVNode,   
            Fragment: _Fragment, openBlock: _openBlock, createBlock: _createBlock} = _Vue  
  
            const _component_component_a = _resolveComponent("component-a")  
            const _component_component_b = _resolveComponent("component-b")  
            const _component_component_c = _resolveComponent("component-c")  
  
            return (_openBlock(),  
            _createBlock(_Fragment, null, [  
              _createVNode(_component_component_a),   
              _createVNode(_component_component_b),   
              _createVNode(_component_component_c)], 64))  
        }  
    }  
})  

```

在获取到组件之后，会通过 `_createVNode` 函数创建 `VNode` 节点。然而，关于 `VNode` 是如何被渲染成真实的 DOM 元素这个过程，就不继续往下介绍了，后续会写专门的文章来单独介绍这块的内容，接下来我们将介绍动态组件的相关内容。

### 二、动态组件

在 Vue 3 中为我们提供了一个 `component` 内置组件，该组件可以渲染一个 “元组件” 为动态组件。根据 `is` 的值，来决定哪个组件被渲染。如果 `is` 的值是一个字符串，它既可以是 HTML 标签名称也可以是组件名称。对应的使用示例如下：

```
<!--  动态组件由 vm 实例的 `componentId` property 控制 -->  
<component :is="componentId"></component>  
  
<!-- 也能够渲染注册过的组件或 prop 传入的组件-->  
<component :is="$options.components.child"></component>  
  
<!-- 可以通过字符串引用组件 -->  
<component :is="condition ? 'FooComponent' : 'BarComponent'"></component>  
  
<!-- 可以用来渲染原生 HTML 元素 -->  
<component :is="href ? 'a' : 'span'"></component>  

```

#### 2.1 绑定字符串类型

介绍完 `component` 内置组件，我们来举个简单的示例：

```
<div id="app">  
   <button  
      v-for="tab in tabs"  
      :key="tab"  
      @click="currentTab = 'tab-' + tab.toLowerCase()">  
      {{ tab }}  
   </button>  
   <component :is="currentTab"></component>  
</div>  
<script> const { createApp } = Vue  
   const tabs = ['Home', 'My']  
   const app = createApp({  
     data() {  
       return {  
         tabs,  
         currentTab: 'tab-' + tabs[0].toLowerCase()  
       }  
     },  
   });  
   app.component('tab-home', {  
     template: `<div style="border: 1px solid;">Home component</div>`  
   })  
   app.component('tab-my', {  
     template: `<div style="border: 1px solid;">My component</div>`  
   })  
   app.mount('#app')</script>  

```

在以上代码中，我们通过 `app.component` 方法全局注册了 `tab-home` 和 `tab-my` 2 个组件。此外，在模板中，我们使用了 `component` 内置组件，该组件的 `is` 属性绑定了 `data` 对象的 `currentTab` 属性，该属性的类型是字符串。当用户点击 Tab 按钮时，会动态更新 `currentTab` 的值，从而实现动态切换组件的功能。以上示例成功运行后的结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V3avmCJr5XatqAuVMKMqVTHACCSxtHiazASaQ78fs2pIwd2Pxp3scOPLZIg2dAEsCC62kLBQiaib1tww/640?wx_fmt=jpeg)

看到这里你会不会觉得 `component` 内置组件挺神奇的，感兴趣的小伙伴继续跟一起，来揭开它背后的秘密。下面我们利用 **Vue 3 Template Explorer** 在线工具，看一下 `<component :is="currentTab"></component>` 模板编译的结果：

```
const _Vue = Vue  
  
return function render(_ctx, _cache, $props, $setup, $data, $options) {  
  with (_ctx) {  
    const { resolveDynamicComponent: _resolveDynamicComponent, openBlock: _openBlock,   
      createBlock: _createBlock } = _Vue  
    return (_openBlock(), _createBlock(_resolveDynamicComponent(currentTab)))  
  }  
}  

```

通过观察生成的渲染函数，我们发现了一个 `resolveDynamicComponent` 的函数，根据该函数的名称，我们可以知道它用于解析动态组件，它被定义在 `runtime-core/src/helpers/resolveAssets.ts` 文件中，具体实现如下所示：

```
// packages/runtime-core/src/helpers/resolveAssets.ts  
export function resolveDynamicComponent(component: unknown): VNodeTypes {  
  if (isString(component)) {  
    return resolveAsset(COMPONENTS, component, false) || component  
  } else {  
    // invalid types will fallthrough to createVNode and raise warning  
    return (component || NULL_DYNAMIC_COMPONENT) as any  
  }  
}  

```

在 `resolveDynamicComponent` 函数内部，若 `component` 参数是字符串类型，则会调用前面介绍的 `resolveAsset` 方法来解析组件：

```
// packages/runtime-core/src/helpers/resolveAssets.ts  
function resolveAsset( type: typeof COMPONENTS | typeof DIRECTIVES,  
  name: string,  
  warnMissing = true) {  
  const instance = currentRenderingInstance || currentInstance  
  if (instance) {  
    const Component = instance.type  
    // 省略大部分处理逻辑  
    const res =  
      // 局部注册  
      // check instance[type] first for components with mixin or extends.  
      resolve(instance[type] || (Component as ComponentOptions)[type], name) ||  
      // 全局注册  
      resolve(instance.appContext[type], name)  
    return res  
  }  
}  

```

对于前面的示例来说，组件是全局注册的，所以解析过程中会从 `app.context` 上下文对象的 `components` 属性中获取对应的组件。当 `currentTab` 发生变化时，`resolveAsset` 函数就会返回不同的组件，从而实现动态组件的功能。

此外，如果 `resolveAsset` 函数获取不到对应的组件，则会返回当前 `component` 参数的值。比如 `resolveDynamicComponent('div')` 将返回 `'div'` 字符串。

```
// packages/runtime-core/src/helpers/resolveAssets.ts  
export const NULL_DYNAMIC_COMPONENT = Symbol()  
  
export function resolveDynamicComponent(component: unknown): VNodeTypes {  
  if (isString(component)) {  
    return resolveAsset(COMPONENTS, component, false) || component  
  } else {  
    return (component || NULL_DYNAMIC_COMPONENT) as any  
  }  
}  

```

细心的小伙伴可能也注意到了，在 `resolveDynamicComponent` 函数内部，如果 `component` 参数非字符串类型，则会返回 `component || NULL_DYNAMIC_COMPONENT` 这行语句的执行结果，其中 `NULL_DYNAMIC_COMPONENT` 的值是一个 Symbol 对象。

#### 2.2 绑定对象类型

了解完上述的内容之后，我们来重新实现一下前面动态 Tab 的功能：

```
<div id="app">  
   <button  
      v-for="tab in tabs"  
      :key="tab"  
      @click="currentTab = tab">  
     {{ tab.name }}  
   </button>  
   <component :is="currentTab.component"></component>  
</div>  
<script> const { createApp } = Vue  
   const tabs = [  
     {  
       name: 'Home',  
       component: {  
         template: `<div style="border: 1px solid;">Home component</div>`  
       }  
     },  
     {  
       name: 'My',  
       component: {  
         template: `<div style="border: 1px solid;">My component</div>`  
       }  
   }]  
   const app = createApp({  
     data() {  
       return {  
         tabs,  
         currentTab: tabs[0]  
       }  
     },  
   });  
   app.mount('#app')</script>  

```

在以上示例中，`component` 内置组件的 `is` 属性绑定了 `currentTab` 对象的 `component` 属性，该属性的值是一个对象。当用户点击 Tab 按钮时，会动态更新 `currentTab` 的值，导致 `currentTab.component` 的值也发生变化，从而实现动态切换组件的功能。**需要注意的是，每次切换的时候，都会重新创建动态组件**。但在某些场景下，你会希望保持这些组件的状态，以避免反复重渲染导致的性能问题。

对于这个问题，我们可以使用 Vue 3 的另一个内置组件 —— `keep-alive`，将动态组件包裹起来。比如：

```
<keep-alive>  
   <component :is="currentTab"></component>  
</keep-alive>
```

`keep-alive` 内置组件的主要作用是用于保留组件状态或避免重新渲染，使用它包裹动态组件时，会缓存不活动的组件实例，而不是销毁它们。关于 `keep-alive` 组件的内部工作原理，后面会写专门的文章来分析它，对它感兴趣的小伙伴记得关注 **Vue 3.0 进阶** 系列哟。

### 三、有话说

#### 3.1 除了 component 内置组件外，还有哪些内置组件？

在 Vue 3 中除了本文介绍的 `component` 和 `keep-alive` 内置组件之外，还提供了 `transition`、`transition-group` 、`slot` 和 `teleport` 内置组件。

#### 3.2 注册全局组件与局部组件有什么区别？

##### 注册全局组件

```
const { createApp, h } = Vue  
const app = createApp({});  
app.component('component-a', {  
  template: "<p>我是组件A</p>"  
});  

```

使用 `app.component` 方法注册的全局的组件，被保存到 `app` 应用对象的上下文对象中。而通过组件对象 `components` 属性注册的局部组件是保存在组件实例中。

##### 注册局部组件

```
const { createApp, h } = Vue  
const app = createApp({});  
const componentA = () => h('div', '我是组件A');  
app.component('component-b', {  
  components: {  
    'component-a': componentA  
  },  
  template: `<div>  
    我是组件B，内部使用了组件A  
    <component-a></component-a>      
  </div>`  
})  
```

##### 解析全局注册和局部注册的组件

```
// packages/runtime-core/src/helpers/resolveAssets.ts  
function resolveAsset( type: typeof COMPONENTS | typeof DIRECTIVES,  
  name: string,  
  warnMissing = true) {  
  const instance = currentRenderingInstance || currentInstance  
  if (instance) {  
    const Component = instance.type  
    // 省略大部分处理逻辑  
    const res =  
      // 局部注册  
      // check instance[type] first for components with mixin or extends.  
      resolve(instance[type] || (Component as ComponentOptions)[type], name) ||  
      // 全局注册  
      resolve(instance.appContext[type], name)  
    return res  
  }  
}  

```

#### 3.3 动态组件能否绑定其他属性？

`component` 内置组件除了支持 `is` 绑定之外，也支持其他属性绑定和事件绑定：

```
<component :is="currentTab.component" :name="name" @click="sayHi"></component>  
```

这里阿宝哥使用 Vue 3 Template Explorer 这个在线工具，来编译上述的模板：

```
const _Vue = Vue  
return function render(_ctx, _cache, $props, $setup, $data, $options) {  
  with (_ctx) {  
    const { resolveDynamicComponent: _resolveDynamicComponent,   
      openBlock: _openBlock, createBlock: _createBlock } = _Vue  
  
    return (_openBlock(), _createBlock(_resolveDynamicComponent(currentTab.component), {  
      name: name,  
      onClick: sayHi  
    }, null, 8 /* PROPS */, ["name", "onClick"]))  
  }  
}  

```

观察以上的渲染函数可知，除了 `is` 绑定会被转换为 `_resolveDynamicComponent` 函数调用之外，其他的属性绑定都会被正常解析为 `props` 对象。

### 四、参考资源

*   Vue 3 官网 - 应用 API
    
*   Vue 3 官网 - 内置组件
