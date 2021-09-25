# Vue源码-响应式

## 简介

本篇文章属于新手向，着重点在理解Vue响应式系统中的Observer、Dep、Watcher,这是理解响应式源码的关键。因此这篇文章不会在分析源码上花很多功夫。



## Object.defineProperty

我们都知道Vue的响应式原理是通过`Object.defineProperty`修改get和set方法来实现的，当我们访问一个变量时就会执行get方法，当我们修改一个变量时会执行set方法。为了实现响应式，我们可以在get方法中收集依赖，这样就能知道哪些数据依赖当前数据。当数据发生变化时，set方法通知依赖者(watcher)进行数据更新。

Observer就是给对象的属性添加 getter 和 setter，用于依赖收集和派发更新。

在介绍observer前，先要了解initState()的过程。



## initState

在new Vue时，会执行`_init`方法进行初始化，当中有一个`initState`方法，主要是对 `props`、`methods`、`data`、`computed` 和 `watcher` 等属性做了初始化操作。这里我们重点分析 `props` 和 `data`。

- initProps
  - 遍历所有props，调用 `defineReactive` 方法把每个 `prop` 对应的值变成响应式。
  - 通过 `proxy` 把 `vm._props.xxx` 的访问代理到 `vm.xxx` 上。
- initData
  - 调用 `observe` 方法观测整个 `data` 的变化，把 `data` 也变成响应式。
  - 通过 `proxy` 把每一个值 `vm._data.xxx` 都代理到 `vm.xxx` 上。

这里涉及三个陌生的方法：`proxy` 、`observe`和`defineReactive`，下面分别介绍。

- `proxy`：

  ```js
  export function proxy (target: Object, sourceKey: string, key: string) {
    sharedPropertyDefinition.get = function proxyGetter () {
      return this[sourceKey][key]
    }
    sharedPropertyDefinition.set = function proxySetter (val) {
      this[sourceKey][key] = val
    }
    Object.defineProperty(target, key, sharedPropertyDefinition)
  }
  ```

简单来讲，创建一个`vm.xxx`属性，当我们访问和修改这个属性时，其实是在访问和修改`vm._data.xxx`，这是就是为什么我们能够通过this来访问到data和props。

- `observe`：

`observe` 方法的作用就是给非 VNode 的对象类型数据添加一个 `Observer`，如果已经添加过则直接返回，否则在满足一定条件下去实例化一个 `Observer` 对象实例，Observer等一会介绍。

- `defineReactive`：

`defineReactive` 的功能就是定义一个响应式对象，给对象动态添加 getter 和 setter，它的定义在 `src/core/observer/index.js` 中。

步骤：

1. new Dep()。 
2. 拿到 `obj` 的属性描述符，然后对子对象递归调用 `observe` 方法。



## Observer

Observer步骤：

1. new Dep()。
2. 执行 `def` 函数把自身实例添加到数据对象 `value` 的 `__ob__` 属性上。
3. 对数据`value`进行判断，如果是数组则遍历调用`observe`方法，如果是对象则调用 `walk` 方法，而 `walk` 方法是遍历对象的 key 调用 `defineReactive` 方法，那么我们来看一下这个方法是做什么的。

```js
/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that has this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  /**
   * Walk through each property and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }
}
```



## 依赖收集

### Dep

```javascript
export default class Dep {
  static target: ?Watcher; // 静态属性target(Watcher类型)
  id: number;
  subs: Array<Watcher>; // 一个（Watcher类型）数组
  
  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    /* 为后续数据变化时候能通知到哪些 subs 做准备 */
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    /* 所有Watcher的实例对象 */
    const subs = this.subs.slice()
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

```

什么是依赖？

```html
<div>
    <p>{{message}}</p>
</div>
```

```javascript
data: {
    text: 'hello world',
    message: 'hello vue',
}

```

```javascript
watch: {
    message: function (val, oldVal) {
        console.log('new: %s, old: %s', val, oldVal)
    },
}   
```

```javascript
computed: {
    messageT() {
        return this.message + '!';
    }
}
```

这里可以看到，虽然data中有text和message，但只有message被使用了，因此只有message需要收集依赖。

![](https://user-gold-cdn.xitu.io/2019/6/2/16b1857fd4532ff0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Dep如何收集依赖?

在getter中收集，当数据被访问时调用`dep.depend`。

什么是依赖？

其实就是`watcher`。



### Watcher

```javascript
export default class Watcher {
  vm: Component;
  deep: boolean;
  sync: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    this.vm = vm
    vm._watchers.push(this)
    /* 监听器的options */
    if (options) {
      this.deep = !!options.deep // 深度监听
      // ...
      this.sync = !!options.sync // 在当前 Tick 中同步执行 watcher 的回调函数，否则响应式数据发生变化之后，watcher回调会在nextTick后执行；
    }
    /* Watcher实例持有Dep的实例的数组 */
    this.deps = [] // 老的Dep集合
    this.newDeps = [] // 触发更新生成的新的Dep集合
    this.depIds = new Set()
    this.newDepIds = new Set()

  get () {
    /* 收集Watcher实例,也就是Dep.target */
    pushTarget(this)
    let value
    const vm = this.vm
    try {
      /* this.getter对应就是我们上篇讲到的vm._update(vm._render(), hydrating),_update会生成VNode,在这个过程中会访
      问vm上的data，这时候就触发了数据对象的getter，defineReactive中可以发现每个getter都持有一个dep,
      因此在触发getter的时候会触发Dep的depend方法，也就触发了Watcher的addDep方法 */
      value = this.getter.call(vm, vm)
    } finally {
      /* 把 Dep.target 恢复成上一个状态，因为当前 vm 的数据依赖收集已经完成，那么对应的渲染Dep.target 也需要改变 */
      popTarget()
      this.cleanupDeps()
    }
    return value
  }

  addDep (dep: Dep) {
    const id = dep.id
    /* 保证同一数据不会被添加多次 */
    if (!this.newDepIds.has(id)) {
      this.newDepIds.add(id)
      this.newDeps.push(dep)
      if (!this.depIds.has(id)) {
        /* 把当前的 watcher 订阅到这个数据持有的 dep 的 subs, 目的是为后续数据变化时候能通知到哪些 subs 做准备 */
        dep.addSub(this)
      }
    }
  }
}

```

 `watcher`中有两个方法:

1. `addDep`:看看自己依赖谁，然后告诉被依赖者，让自己进入被依赖者的`sub`名单。这样当被依赖者发生改变时就通知自己更新。
2. `update`:当自己需要更新时调用，这个方法会在数据更新后渲染到页面上，并且调用`update`这个声明周期函数。



## 总结

回顾一下，`Vue`响应式原理的核心就是`Observer`、`Dep`、`Watcher`。

`Observer`中进行响应式的绑定，在数据被读的时候，触发`get`方法，执行`Dep`来收集依赖，也就是收集`Watcher`。

在数据被改的时候，触发`set`方法，通过对应的所有依赖(`Watcher`)，去执行更新。比如`watch`和`computed`就执行开发者自定义的回调方法。



本文参考了https://juejin.cn/post/6844903858850758670#heading-1。

