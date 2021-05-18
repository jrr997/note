# Vue源码-组件化

## 简介

组件化是Vue的一个核心思想。所谓组件化，就是把页面拆分成多个组件 (component)，每个组件依赖的 CSS、JavaScript、模板、图片等资源放在一起开发和维护。组件是资源独立的，组件在系统内部可复用，组件和组件之间可以嵌套。

本文主要关注Vue如何创建组件。



## createComponent

在上一篇文章中介绍过`vm._render`方法，作用是根据`render`函数生成VDOM，生成VDOM主要是由`vm_render`方法中的`createElement`中的`_createElement`方法实现的。

`_createElement`中有一段逻辑是对参数 `tag` 的判断，如果是一个普通的 html 标签，如div，则会实例化一个普通 VNode 节点，否则通过 `createComponent` 方法创建一个组件 VNode。

`createComponent`有三个核心步骤：

1. 构造子类构造函数
2. 安装组件钩子函数
3. 实例化 VNode

下面我会分别介绍这三个步骤。



### 构造子类构造函数

```javascript
const baseCtor = context.$options._base

// plain options object: turn it into a constructor
if (isObject(Ctor)) {
  Ctor = baseCtor.extend(Ctor)
}
```

这里 `baseCtor` 实际上就是 Vue，这个的定义是在最开始初始化 Vue 的阶段，在 `src/core/global-api/index.js` 中的 `initGlobalAPI` 函数有这么一段逻辑：

```js
// this is used to identify the "base" constructor to extend all plain-object
// components with in Weex's multi-instance scenarios.
Vue.options._base = Vue
```

我们可以注意到`context.$options._base`和`Vue.options._base`，前者是实例中取出`_base`，后者是构造函数中取出`_base`,那么为什么这两个地方都能取出`_base`呢？

实际上在 `src/core/instance/init.js` 里 Vue 原型上的 `_init` 函数中有这么一段逻辑：

```js
vm.$options = mergeOptions(
  resolveConstructorOptions(vm.constructor),
  options || {},
  vm
)
```

这样就把 Vue 上的一些 `option` 扩展到了 vm.$options 上，意思是vm(组件)的`options`中包含了Vue的`options`和用户传入的`options`。

回到`Ctor = baseCtor.extend(Ctor)`，这里Ctor是Vue的子类，继承了Vue的属性和方法，extend返回的是一个叫`Sub`的组件构造函数。当我们去实例化`Sub`时，就会执行 `this._init` 逻辑再次走到了 `Vue` 实例的初始化逻辑，这意味着组件的实例化和Vue的实例化相类似。



### 安装钩子函数

```js
// install component management hooks onto the placeholder node
installComponentHooks(data)
```

整个 `installComponentHooks` 的过程就是把 `componentVNodeHooks` 的钩子函数合并到 `data.hook` 中，在 VNode 执行 `patch` 的过程中执行相关的钩子函数。

简单地说是把生成VNode的方法都准备好，准备生成Vnode。



### # 实例化VNode

```js
const name = Ctor.options.name || tag
const vnode = new VNode(
  `vue-component-${Ctor.cid}${name ? `-${name}` : ''}`,
  data, undefined, undefined, undefined, context,
  { Ctor, propsData, listeners, tag, children },
  asyncFactory
)
return vnode
```

最后一步非常简单，通过 `new VNode` 实例化一个 `vnode` 并返回。需要注意的是和普通元素节点的 `vnode` 不同，组件的 `vnode` 是没有 `children` 的，这点很关键，在之后的 `patch` 过程中我们会再提。



### 总结

创建一个组件经历了三个步骤：

1. 创建组件的构造函数
2. 执行构造函数，准备好生成Vnode的相关方法。
3. 生成Vnode。

下图是组件创建过程的函数调用栈，通过调用栈我们也能更好地了解组件的创建过程。

![组件创建过程](C:\Users\Jrrr\Desktop\Chrome Download\组件创建过程.png)

由下往上看，跟着函数调用栈的顺序，可以看出整个过程是

- 执行 new Vue，然后初始化 vm 实例，扩展原型方法
- 执行 mount 函数去解析编译 template 模板
- 执行 Watcher render 去生成 VNode tree
- 执行 update 中的 patch 去生成 DOM，如遇到组件

- - 初始化 Sub 构造器
  - 执行 new Sub，初始化组件 vm 实例，扩展原型方法
  - 执行 mount 函数解析编译组件模板，生成组件 VNode tree，最终返回组件 DOM
  - 如再遇组件，则继续递归

可以看出 Vue 的组件生成顺序是由子到父的。

## Patch

既然有了VNode，下一步应该是调用`vm._update`来创建真实DOM，这个过程就是patch。

具体的过程就不细说，这里主要注意一个组件中可能又包含另外的组件。如果组件 `patch` 过程中又创建了子组件，那么DOM 的插入顺序是先子后父。



## 组件注册

### 全局注册

相当于在`Vue.options.components`注册了一个组件。因为局部组件都会通过`Vue.extend`创建一个`Sub`子类(前面介绍过)，而`Sub`会继承`Vue.options.components`，这里是通过`mergeOptions`实现的，前面也简单提到过。

到这里，局部组件就能通过`options`拿到全局组件。



### 局部注册

Vue.js 也同样支持局部注册，我们可以在一个组件内部使用 `components` 选项做组件的局部注册，例如：

```js
import HelloWorld from './components/HelloWorld'

export default {
  components: {
    HelloWorld
  }
}
```

其实理解了全局注册的过程，局部注册是非常简单的。在组件的 Vue 的实例化阶段有一个合并 `option` 的逻辑，之前我们也分析过，所以就把 `components` 合并到 `vm.$options.components` 上，这样我们就可以在 `resolveAsset` 的时候拿到这个组件的构造函数，并作为 `createComponent` 的钩子的参数。

注意，局部注册和全局注册不同的是，只有该类型的组件才可以访问局部注册的子组件，而全局注册是扩展到 `Vue.options` 下，所以在所有组件创建的过程中，都会从全局的 `Vue.options.components` 扩展到当前组件的 `vm.$options.components` 下，这就是全局注册的组件能被任意使用的原因。



## 异步组件

因为我在实战中并没有用过异步组件，这里就先留个坑，只是简单的了解一下异步组件的作用。

在我们平时的开发工作中，为了减少首屏代码体积，往往会把一些非首屏的组件设计成异步组件，按需加载。Vue 也原生支持了异步组件的能力，如下：

```js
Vue.component('async-example', function (resolve, reject) {
   // 这个特殊的 require 语法告诉 webpack
   // 自动将编译后的代码分割成不同的块，
   // 这些块将通过 Ajax 请求自动下载。
   require(['./my-async-component'], resolve)
})
```



## 生命周期

生命周期是一个很重要的知识点，我要单独开一篇文章来讲。