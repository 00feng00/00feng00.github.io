---
layout:       post
title:        "vue3 依赖注入"
subtitle:     "vue3 provide inject"
date:         2021-03-03 22:05:00
author:       "ZeFeng"
header-img:   "img/websocket_logo.jpg"
header-mask:  0.3
catalog:      true
tags:
    - VUE3
    - 源码
---

本文是 **Vue 3.0 进阶系列** 的第六篇文章，在这篇文章中，将带大家一起探索 Vue 3 中的依赖注入功能。

使用过 Angular 的小伙伴对 **依赖注入** 应该不会陌生，依赖注入简称为 DI（Dependency Injection）。组件之间的依赖关系由容器在运行期决定，形象的说，即由容器动态的将某个依赖关系注入到组件之中。**依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。**


在 Vue 3.0 中，为我们提供了简单的依赖注入功能 —— **provide/inject**。它们解决了以下问题：有一些深度嵌套的组件，而深层的子组件只需要父组件的部分内容。在这种情况下，如果仍然将 prop 沿着组件链逐级传递下去，这样使用起来会很麻烦。

为了解决上述问题，Vue 提供了 `provide` 和 `inject`。使用 **provide/inject** 之后，无论组件层次结构有多深，父组件都可以作为其所有子组件的依赖提供者。

![](https://mmbiz.qpic.cn/mmbiz_png/jQmwTIFl1V0mwU2nOTonxKBUrGyc1dWCC5ibxWH8KRjSvkxkAPibpZQMZQicyUaGPDyAH0BCchymFM4EAAPRibLsIw/640?wx_fmt=png)

（图片来源 —— https://v3.cn.vuejs.org/guide/component-provide-inject.html）

由上图可知，在父组件上通过 `provide` 提供数据，子组件通过 `inject` 注入数据。介绍完 **provide/inject** 的作用之后，我们来看一下具体的示例。

### 一、Provide/Inject 使用示例

在使用 **provide/inject** 特性时，你可以通过 **provide** 和 **inject** 选项的方式来使用它。这种方式官方文档已经有介绍了，这里将介绍另一种使用方式，即在组合式 API 的 `setup` 组件选项中，通过 `provide` 和 `inject` 函数的方式来实现依赖注入。

```
<div id="app"></div>  
<script> const { createApp, h, provide, inject } = Vue  
   const app = createApp({  
     render: () => h(Provider)  
   })  
     
   const Provider = {  
     setup() {  
       provide('name', '啊啊啊')  
       return () => h(Middle)  
     }  
   }  
  
   const Middle = {  
     render: () => h(Consumer)  
   }  
  
   const Consumer = {  
     setup() {  
       const name = inject('name')  
       return () => `大家好，我是${name}!`  
     }  
   }  
   app.mount('#app') </script>  
```

在以上示例中，在 `Provider` 组件内通过 `provide` 函数配置了数据，而在 `Consumer` 组件中通过 `inject` 函数获取 `Provider` 组件中已配置的数据。需要注意的是，示例中的 `Consumer` 组件是作为 `Provider` 组件的孙组件。因此，通过使用 **provide/inject** 提供的依赖注入功能，我们实现了数据的跨层级传递。

介绍完 `provide` 和 `inject` 函数的基本使用之后，接下来将带大家一起来揭开它们背后的秘密。

### 二、Provide 函数

在分析 `provide` 函数之前，我们先来回顾一下它的用法：

```
const Provider = {  
  setup() {  
    provide('name', '啊啊啊')  
    return () => h(Middle)  
  }  
}  

```

该函数被定义在 `runtime-core/src/apiInject.ts` 文件中：

```
// packages/runtime-core/src/apiInject.ts  
export interface InjectionKey<T> extends Symbol {}  
  
export function provide<T>(key: InjectionKey<T> | string | number, value: T) {  
  if (!currentInstance) {  
    if (__DEV__) {  
      warn(`provide() can only be used inside setup().`)  
    }  
  } else {  
    let provides = currentInstance.provides  
    // 默认情况下，组件实例会继承于它父组件实例的 provides 对象，当它需要为本身提供  
    // 值是，它将使用父组件的 provides 对象作为原型对象来创建属于它自己的 provides  
    // 对象。这样的话，`inject` 函数就可以简单地从直接父对象中查找需注入的值，并让原型链  
    // 来完成这个工作。  
    const parentProvides =  
      currentInstance.parent && currentInstance.parent.provides  
    if (parentProvides === provides) {  
      provides = currentInstance.provides = Object.create(parentProvides)  
    }  
    // TS 不允许使用 symbol 作为索引类型  
    provides[key as string] = value  
  }  
}  
```

通过观察以上的代码，我们可以得出以下结论：

*   `provide` 函数只能使用在组合式 API 的 `setup` 函数中。
    
*   组件实例上会有一个 `provides` 属性，通过 `provide` 配置的数据，最终会被保存到组件的 `provides` 属性中。
    
*   `provide` 函数支持 3 种类型作为 `key`，即 `InjectionKey<T> | string | number`，其中 `InjectionKey` 类型是 `Symbol` 类型的子类型。
    

在以上代码中，我们见到了 `currentInstance` 对象，那么这个对象内部的结构是怎样的呢？为了一探究竟，我们在 `provide` 函数内部加个断点：

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V0mwU2nOTonxKBUrGyc1dWC61Deu0mviapgN3z2n2jmBhBJMF631bGOjBcicRXscsybWtKEibia4sciaAA/640?wx_fmt=jpeg)

由上图可知， `currentInstance` 是一个含有多种属性的普通对象。其中 `bc（BEFORE_CREATE）`、`bm（BEFORE_MOUNT）` 和 `bu（BEFORE_UPDATE）` 是与生命周期相关的钩子。那么问题又来了，`currentInstance` 对象是怎么创建的？看过之前 Vue 3.0 进阶系列文章的小伙伴，可能对 `createComponentInstance` 函数有点印象，该函数的作用就是创建组件实例，具体代码如下所示：

```
// packages/runtime-core/src/component.ts  
export function createComponentInstance( vnode: VNode,  
  parent: ComponentInternalInstance | null,  
  suspense: SuspenseBoundary | null) {  
  const type = vnode.type as ConcreteComponent  
  // inherit parent app context - or - if root, adopt from root vnode  
  const appContext =  
    (parent ? parent.appContext : vnode.appContext) || emptyAppContext  
  
  const instance: ComponentInternalInstance = {  
    uid: uid++, vnode, type,  
    parent, appContext,  
    root: null!, // to be immediately set  
    subTree: null!, // will be set synchronously right after creation  
    update: null!, // will be set synchronously right after creation  
    render: null, effects: null,  
    provides: parent ? parent.provides : Object.create(appContext.provides),  
    bc, bm, bu  
    // 省略大部分属性  
  }  
  // 省略部分代码  
  instance.root = parent ? parent.root : instance  
  instance.emit = emit.bind(null, instance)  
  return instance  
}  
```

需要注意的是，如果当前组件 `parent` 属性的值不为 `null` 时，则将使用 `parent.provides` 的值作为组件实例 `provides` 属性的属性值。介绍完 `provide` 和 `createComponentInstance` 函数，为了让大家能够更好地理解前面的示例，用一张图来总结一下示例中组件之间的关系：

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V0mwU2nOTonxKBUrGyc1dWCib1kCBUHmicAbBicibwTBNqiaDcibyOyVa9hN6G8BibpBP7DVTXYtpUcia2z7w/640?wx_fmt=jpeg)

对于根组件来说，它的 `parent` 属性值为 `null`。好的，`provide` 函数就先介绍到这里，下面我们来开始介绍 `inject` 函数。

### 三、Inject 函数

同样，在分析 `inject` 函数之前，我们也先来回顾一下它的用法：

```
const Consumer = {  
  setup() {  
    const name = inject('name')  
    return () => `大家好，我是${name}!`  
  }  
}  
```

`inject` 函数与 `provide` 函数是互相配合的，它们都被定义在 `runtime-core/src/apiInject.ts` 文件中：

```
// packages/runtime-core/src/apiInject.ts  
export function inject( key: InjectionKey<any> | string,  
  defaultValue?: unknown,  
  treatDefaultAsFactory = false) {  
  const instance = currentInstance || currentRenderingInstance // 获取当前实例  
  if (instance) {  
    // to support `app.use` plugins,  
    // fallback to appContext's `provides` if the intance is at root  
    const provides =  
      instance.parent == null  
        ? instance.vnode.appContext && instance.vnode.appContext.provides  
        : instance.parent.provides  
  
    if (provides && (key as string | symbol) in provides) {  
      // TS doesn't allow symbol as index type  
      return provides[key as string]  
    } else if (arguments.length > 1) {  
      return treatDefaultAsFactory && isFunction(defaultValue)  
        ? defaultValue()  
        : defaultValue  
    } else if (__DEV__) {  
      warn(`injection "${String(key)}" not found.`)  
    }  
  } else if (__DEV__) {  
    warn(`inject() can only be used inside setup() or functional components.`)  
  }  
}  
```

在 `inject` 函数中，我们可以清楚地看到如果当前实例的 `parent` 属性为 `null` 时，则会从 `appContext` 上下文中获取 `provides` 对象，否则将从当前实例的父组件实例中获取 `provides` 对象。

对于我们的示例来说，在获取到 `provides` 对象后，就会判断 `name` 属性是否存在于当前的 `provides` 对象中，此时该对象是 `{ name: "啊啊啊"}`，所以会直接返回 **"啊啊啊"**。

![](https://mmbiz.qpic.cn/mmbiz_jpg/jQmwTIFl1V0mwU2nOTonxKBUrGyc1dWCszpbZ7HmS3QJBTsyqicibjEibUwDHzFVElWj7CSkQ6tQcHUHFweiaFo2og/640?wx_fmt=jpeg)

此外，通过观察 `inject` 函数，我们还可以得出以下结论：

*   `inject` 函数的第二个参数是一个可选参数 —— `defaultValue?: unknown`，用于设置默认值或默认值工厂。即在 `provides` 对象上找不到 `key` 对应值的时候，可以使用默认值或默认值工厂的返回值来代替。
    
*   `inject` 函数只能在 `setup()` 或函数组件中使用。
    

### 四、App 对象中的 provide API

在创建完 Vue 3 应用对象之后，我们可以使用该对象提供的 `provide` 方法。该方法设置一个可以被注入到应用范围内所有组件中的值。之后，组件就可以使用 `inject` 来接收 `provide` 的值。

```
import { createApp } from 'vue'  
  
const app = createApp({  
  inject: ['name'],  
  template: `  
    <div>  
      {{ name }}  
    </div>  
  `  
})  
  
app.provide('name', '阿宝哥')  
```

需要注意的是，`app.provide` 方法不应该与 `provide` 组件选项或组合式 API 中的 `provide` 方法混淆。虽然它们也是相同的 **provide/inject** 机制的一部分，但是是用来配置组件 `provide` 的值而不是应用 `provide` 的值。

介绍完 `app.provide` 方法之后，我们来了解一下它的实现。看过 **”Vue 3.0 进阶“** 系列教程的小伙伴，对 `app` 对象应该不会陌生。因为在前面的文章中，阿宝哥已经介绍过 `component`、`directive` 和 `mount` 等方法。接下来，我们来看一下 `provide` 方法的具体实现：

```
// packages/runtime-core/src/apiCreateApp.ts  
export function createAppAPI<HostElement>( render: RootRenderFunction,  
  hydrate?: RootHydrateFunction): CreateAppFunction<HostElement> {  
  return function createApp(rootComponent, rootProps = null) {  
    const context = createAppContext()  
  
    // 省略大部分内容  
    const app: App = (context.app = {  
      _uid: uid++,  
      _context: context,  
  
      provide(key, value) {  
        // TypeScript doesn't allow symbols as index type  
        // https://github.com/Microsoft/TypeScript/issues/24587  
        context.provides[key as string] = value  
        return app  
      }  
    })  
  
    return app  
  }  
}  
```

由以上代码可知，在 `provide` 方法内部会把 `key` 和 `value` 以键值对的形式保存在应用上下文 `context` 对象的 `provides` 属性中。

```
// packages/runtime-core/src/apiCreateApp.ts  
export function createAppContext(): AppContext {  
  return {  
    app: null as any,  
    config: { ... },  
    mixins: [],  
    components: {},  
    directives: {},  
    provides: Object.create(null)  
  }  
}  
```

### 五、阿宝哥有话说

#### 5.1 在嵌套的 providers 场景下，存在同名的 key 会怎么样？

```
<div id="app"></div>  
<script> const { createApp, h, provide, inject } = Vue  
   const app = createApp({  
     render: () => h(ProviderOne)  
   })  
     
   const ProviderOne = {  
     setup() {  
       provide('foo', 'foo')  
       provide('bar', 'bar')  
       return () => h(ProviderTwo)  
     }  
   }  
  
   const ProviderTwo = {  
     setup() {  
       provide('foo', 'fooOverride')  
       provide('baz', 'baz')  
       return () => h(Consumer)  
      }  
   }  
  
   const Consumer = {  
     setup() {  
       const foo = inject('foo')  
       const bar = inject('bar')  
       const baz = inject('baz')  
       return () => [foo, bar, baz].join(',')  
     }  
   }  
   app.mount('#app') </script>  
```

在以上代码中，`ProviderOne` 和 `ProviderTwo` 组件中使用同样的 `foo` 属性名配置了 `Provider`。然后，我们在底层的 `Consumer` 组件中使用 `inject` API 分别注入了 `ProviderOne` 和  `ProviderTwo` 中配置的值。接下来，我们先来看一下结果：

```
fooOverride,bar,baz  
```

由以上结果可知，在嵌套 providers 的场景中，会就近从父组件实例中获取对应的值，找不到的话，会往上一层层进行查找。

#### 5.2 通过 inject 获取响应式的值，能否正常工作？

在某些场景下，我们希望往深层的子组件传递通过响应式 API 创建的响应式的值，那么通过 `inject` 函数获取的响应式的值可以正常工作么？要弄清楚这个问题，我们来看一个具体的示例：

```
<div id="app"></div>  
<script>  
   const { createApp, h, provide, inject, ref, onMounted } = Vue  
   const nameRef = ref("阿宝哥");  
   const app = createApp({  
     setup() {  
       provide("name", nameRef);  
       return () => h(Middle)  
     }  
    })  
  
   const Middle = {  
     render: () => h(Consumer)  
   }  
  
   const Consumer = {  
     setup() {  
       const name = inject('name')  
       onMounted(() => {  
         setTimeout(() => nameRef.value = "kakuqo", 2000);  
       })  
       return () => `大家好，我是${name.value}!`  
     }  
  }  
  app.mount('#app')   
</script>  
```

在以上代码中，我们通过 `ref` API 创建了一个 `nameRef` 对象，然后在根组件中通过 `provide` 函数配置相应的 `Provider`。而在 `Consumer` 组件的 `setup` 方法内，我们通过 `inject` 函数注入了 `nameRef` 对象，并通过 `name.value` 访问了该对象内保存的值。

此外，在 `setup` 方法内部，我们还使用了 `onMounted` 生命周期钩子，在钩子对应的回调函数中，我们延迟 2S 修改 `nameRef` 对象的值。

以上示例成功运行后，首先会先显示 **大家好，我是阿宝哥!**，差不多 2S 后页面会刷新为 **大家好，我是kakuqo!**。

#### 5.3 是否支持 self-inject？

什么是 `self-inject` 呢？这里阿宝哥不做过多解释，我们直接来看个具体的例子：

```
<div id="app"></div>  
<script>  
   const { createApp, h, provide, inject } = Vue  
   const app = createApp({  
     render: () => h(Provider)  
   })  
     
   const Provider = {  
     setup() {  
       provide('name', '阿宝哥')  
       const injectedName = inject('name')  
       return () => h(injectedName)  
     }  
   }  
   app.mount('#app')   
</script>  
```

在以上代码中，我们在 `Provider` 组件的 `setup` 方法内部先使用 `provide` 函数配置了相应的 `Provider`，然后使用 `inject` 函数来获取对应的值。很明显这个操作并没有实际的意义，那么可以这样使用么？答案是可以的，以上示例成功运行之后，`Provider` 组件会被转换为 **注释节点**。

```
<div id="app" data-v-app=""><!----></div>  
```

那么为什么会转为注释节点呢？因为 `injectedName` 的值为 `undefined`，在通过 `h` 函数创建 `VNode` 对象的时候，会继续调用 `createVNode` 函数，在该函数内部如果发现是 `type` 类型为 `falsy` 值时，会把 `VNode` 对象的类型统一转换为 `Comment` 类型。

```
// packages/runtime-core/src/vnode.ts  
function _createVNode( type: VNodeTypes | ClassComponent | typeof NULL_DYNAMIC_COMPONENT,  
  props: (Data & VNodeProps) | null = null,  
  children: unknown = null,  
  patchFlag: number = 0,  
  dynamicProps: string[] | null = null,  
  isBlockNode = false): VNode {  
  if (!type || type === NULL_DYNAMIC_COMPONENT) {  
    if (__DEV__ && !type) {  
      warn(`Invalid vnode type when creating vnode: ${type}.`)  
    }  
    type = Comment  
  }  
  // 省略大部分代码  
}  
```

本文阿宝哥主要介绍了依赖注入的概念及作用、如何使用 Vue 3 提供的 **provide/inject** 特性。为了让大家能够更深入地理解 **provide/inject** 特性，阿宝哥从源码角度分析了 `provide` 和 `inject` 函数的具体实现。在后续的文章中，阿宝哥将会介绍在插件中如何应用 **provide/inject** 特性，感兴趣的小伙伴不要错过哟。

### 六、参考资源

*   Vue 3 官网 - Provide/Inject
    
*   Vue 3 官网 - 组合式 API
    
