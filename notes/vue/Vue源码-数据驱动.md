# Vue源码阅读

## 一、vue的源码目录设计

```text
src
├── compiler        # 编译相关 
├── core            # 核心代码 
├── platforms       # 不同平台的支持
├── server          # 服务端渲染
├── sfc             # .vue 文件解析
├── shared          # 共享代码
```

#### compiler

compiler 目录包含 Vue.js 所有编译相关的代码。它包括把模板解析成 ast 语法树，ast 语法树优化，代码生成等功能。

#### core

core 目录包含了 Vue.js 的核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等。



#### Vue入口

Vue入口在

```
src/core/instance/index.js
```

```javascript
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```

这里可以看出Vue是一个构造函数。我们可以看到Vue被传入很多```xxxMixin``` 函数中，这些函数的作用是给Vue.prototype扩展方法。

`initGlobalAPI`

Vue.js 在整个初始化过程中，除了给它的原型 prototype 上扩展方法，还会给 `Vue` 这个对象本身扩展全局的静态方法，它的定义在 `src/core/global-api/index.js` 中，全局方法包括set、delete、nextTick等。



## 二、数据驱动

Vue.js 一个核心思想是数据驱动。数据驱动是指数据驱动视图的生成，我们只用修改数据，视图就会发生相应的修改，免去操作DOM来修改视图的步骤。



#### new Vue 发生了什么？

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}
```

从这里可见，new Vue后执行了this._init方法，并传入了options，而this_init方法主要做了这些事情：合并配置，初始化生命周期，初始化事件中心，初始化渲染，初始化 data、props、computed、watcher 等等。下面是this.\_init的部分代码。

```js
/* istanbul ignore else */
  if (process.env.NODE_ENV !== 'production') {
    initProxy(vm)
  } else {
    vm._renderProxy = vm
  }
  // expose real self
  vm._self = vm
  initLifecycle(vm)
  initEvents(vm)
  initRender(vm)
  callHook(vm, 'beforeCreate')
  initInjections(vm) // resolve injections before data/props
  initState(vm)
  initProvide(vm) // resolve provide after data/props
  callHook(vm, 'created')
```

在beforeCreate前，Vue已经初始化了生命周期、事件和渲染，在beforeCreate和created之间有一个initState方法，initState主要是初始化data、props等属性。意思是beforeCreate不能拿到data、props、methods、computed等，要在created中拿。



#### Vue实例的挂载

`compiler` 版本的 `$mount` 实现非常有意思，先来看一下 `src/platform/web/entry-runtime-with-compiler.js` 文件中定义：

```js
const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && query(el)

  /* istanbul ignore if */
  if (el === document.body || el === document.documentElement) {
    process.env.NODE_ENV !== 'production' && warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
    return this
  }

  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    if (template) {
      if (typeof template === 'string') {
        if (template.charAt(0) === '#') {
          template = idToTemplate(template)
          /* istanbul ignore if */
          if (process.env.NODE_ENV !== 'production' && !template) {
            warn(
              `Template element not found or is empty: ${options.template}`,
              this
            )
          }
        }
      } else if (template.nodeType) {
        template = template.innerHTML
      } else {
        if (process.env.NODE_ENV !== 'production') {
          warn('invalid template option:' + template, this)
        }
        return this
      }
    } else if (el) {
      template = getOuterHTML(el)
    }
    if (template) {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile')
      }

      const { render, staticRenderFns } = compileToFunctions(template, {
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
      }, this)
      options.render = render
      options.staticRenderFns = staticRenderFns

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        mark('compile end')
        measure(`vue ${this._name} compile`, 'compile', 'compile end')
      }
    }
  }
  return mount.call(this, el, hydrating)
}
```

在```runtime-with-compiler```版本的代码中，```const mount = Vue.prototype.$mount```缓存了```runtime-only```的```$mount```，然后重写了```$mount```方法，这个重写的```$mount```方法主要做了两件事情：

第一步：Vue实例通过```$mount ```来挂载vm，且```$mount ```限制Vue实例不能挂载到```body```和```html```根节点上。

第二步：把```el```和```template```字符串转化为```render```函数。Vue 2.0 版本中，所有 Vue 的组件的渲染最终都需要先生成`render` 函数再渲染。

做完这两步之后，会执行之前缓存的```runtime-only```的```$mount```。这个挂载是真正的挂载。

```js
// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```

原先的`$mount` 方法实际上会去调用 `mountComponent` 方法，这个方法定义在 `src/core/instance/lifecycle.js` 文件中：

```js
export function mountComponent (
  vm: Component,
  el: ?Element,
  hydrating?: boolean
): Component {
  vm.$el = el
  if (!vm.$options.render) {
    vm.$options.render = createEmptyVNode
    if (process.env.NODE_ENV !== 'production') {
      /* istanbul ignore if */
      if ((vm.$options.template && vm.$options.template.charAt(0) !== '#') ||
        vm.$options.el || el) {
        warn(
          'You are using the runtime-only build of Vue where the template ' +
          'compiler is not available. Either pre-compile the templates into ' +
          'render functions, or use the compiler-included build.',
          vm
        )
      } else {
        warn(
          'Failed to mount component: template or render function not defined.',
          vm
        )
      }
    }
  }
  callHook(vm, 'beforeMount')

  let updateComponent
  /* istanbul ignore if */
  if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
    updateComponent = () => {
      const name = vm._name
      const id = vm._uid
      const startTag = `vue-perf-start:${id}`
      const endTag = `vue-perf-end:${id}`

      mark(startTag)
      const vnode = vm._render()
      mark(endTag)
      measure(`vue ${name} render`, startTag, endTag)

      mark(startTag)
      vm._update(vnode, hydrating)
      mark(endTag)
      measure(`vue ${name} patch`, startTag, endTag)
    }
  } else {
    updateComponent = () => {
      vm._update(vm._render(), hydrating)
    }
  }

  // we set this to vm._watcher inside the watcher's constructor
  // since the watcher's initial patch may call $forceUpdate (e.g. inside child
  // component's mounted hook), which relies on vm._watcher being already defined
  new Watcher(vm, updateComponent, noop, {
    before () {
      if (vm._isMounted) {
        callHook(vm, 'beforeUpdate')
      }
    }
  }, true /* isRenderWatcher */)
  hydrating = false

  // manually mounted instance, call mounted on self
  // mounted is called for render-created child components in its inserted hook
  if (vm.$vnode == null) {
    vm._isMounted = true
    callHook(vm, 'mounted')
  }
  return vm
}
```

这段代码说明了```mountComponent```做了三件事情：

1. ```callHook(vm, 'beforeMounted')```
2. 调用 `vm._render`生成了VNode，然后调用 `vm._update` 更新 DOM。
3. 创建```Watcher```实例，而```watcher```做又做了两件互斥事情：
   - 一个是初始化的时候会执行回调函数```callHook(vm, 'mounted')```
   - 二是当vm 实例中的监测的数据发生变化的时候执行回调函数```callHook(vm, 'beforeUpdate')``

##### 总结

```callHook(vm, 'beforeMount')```。

1. 模板转化成render函数。
2. `vm._render`生成了VNode。
3. ```callHook(vm, 'beforeUpdate')```
4. `vm._update` 更新 DOM。
5. ```callHook(vm, 'mounted')```或者```callHook(vm, 'beforeUpdate')```



#### vm._render

前面提到`vm._render`,这个方法的作用就是把```render```函数生成VNode，也就是虚拟DOM。

模板：

```html
<div id="app">
  {{ message }}
</div>
```

模板转为```render```函数：

```js
render: function (createElement) {
  return createElement('div', {
     attrs: {
        id: 'app'
      },
  }, this.message)
}
```

```vm._render```方法传入```render```函数，生成VNode，这里的```$createElement```就是创建VNode的方法。

```js
vnode = render.call(vm._renderProxy, vm.$createElement)
```



#### Virtual DOM

首先必须知道为什么需要虚拟DOM？

因为浏览器的DOM是十分“昂贵”的，真实DOM十分庞大，每个真实DOM都有很多属性和方法。如果我们频繁地更新真实DOM，可能会出现性能问题，因此虚拟DOM就出现了。



那么什么是虚拟DOM？

虚拟DOM就是描述真实DOM的JS对象，与真实DOM对比起来，虚拟DOM是轻量的。它的核心定义无非就几个关键属性，标签名、数据、子节点、键值等。

虚拟DOM除了它的数据结构的定义，映射到真实的 DOM 实际上要经历 VNode 的 create、diff、patch 等过程。那么在 Vue.js 中，VNode 的 create 是通过之前提到的 `createElement` 方法创建的，我们接下来分析这部分的实现。



#### createElement

```createElement```做两件事情

1. children 的规范化

   由于 Virtual DOM 实际上是一个树状结构，每一个 VNode 可能会有若干个子节点，这些子节点应该也是 VNode 的类型。`_createElement` 接收的第 4 个参数 children 是任意类型的，因此我们需要把它们规范成 VNode 类型。

   在大多数情况下children都是VNode类型的，但也有例外就是 `functional component` 函数式组件返回的是一个数组而不是一个根节点，所以会通过 `Array.prototype.concat` 方法把整个 `children` 数组打平，让它的深度只有一层。

2. VNode 的创建

   - 单节点就直接创建一个VNode。

   - 如果是多个节点，会用数组表示，遍历创建VNode。
   - 有时候数组中会嵌套数组，这时用递归。

经过这两个步骤就形成了一个 VNode Tree。



回到 `mountComponent` 函数的过程，我们已经知道 `vm._render` 是如何创建了一个 VNode，接下来就是要把这个 VNode 渲染成一个真实的 DOM 并渲染出来，这个过程是通过 `vm._update` 完成的，接下来分析一下这个过程。



#### vm._update

Vue 的 `_update` 是实例的一个私有方法，它被调用的时机有 2 个，一个是首次渲染，一个是数据更新的时候；由于我们这一章节只分析首次渲染部分，数据更新部分会在之后分析响应式原理的时候涉及。`_update` 方法的作用是把 VNode 渲染成真实的 DOM，它的定义在 `src/core/instance/lifecycle.js` 中。

首次渲染时vm._update的主要是调用一个```patch```方法

```js
// initial render
vm.$el = vm.__patch__(vm.$el, vnode, hydrating, false /* removeOnly */)
```

其中```patch```方法又有一个```creatElm```方法，作用是根据虚拟DOM创建真实DOM。



简单地讲，在首次渲染时，`_update`方法根据虚拟DOM树创建一个真实的DOM树然后插入到```vm.$el```上。



## 总结

那么至此我们从主线上把模板和数据如何渲染成最终的 DOM 的过程分析完毕了，我们可以通过下图更直观地看到从初始化 Vue 到最终渲染的整个过程。

![img](https://ustbhuangyi.github.io/vue-analysis/assets/new-vue.png)