# Vue源码-编译

简介：

编译的过程就是把模板变成`render`函数.

Vue.js 提供了 2 个版本，一个是 Runtime + Compiler 的，一个是 Runtime only 的，前者是包含编译代码的，可以把编译过程放在运行时做，后者是不包含编译代码的，需要借助 webpack 的 `vue-loader` 事先把模板编译成 `render`函数。

## 编译过程

vue的编译过程主要分为三步：

- 解析模板字符串生成AST

```js
const ast = parse(template.trim(), options)
```

- 优化语法树

```js
optimize(ast, options)
```

- 生成代码

```js
const code = generate(ast, options)
```



## parse

### 什么是AST

AST是抽象语法树，它具体是这样的：

```html
<ul :class="bindCls" class="list" v-if="isShow">
    <li v-for="(item,index) in data" @click="clickItem(index)">{{item}}:{{index}}</li>
</ul>
```

经过 `parse` 过程后，生成的 AST 如下：

```js
ast = {
  'type': 1,
  'tag': 'ul',
  'attrsList': [],
  'attrsMap': {
    ':class': 'bindCls',
    'class': 'list',
    'v-if': 'isShow'
  },
  'if': 'isShow',
  'ifConditions': [{
    'exp': 'isShow',
    'block': // ul ast element
  }],
  'parent': undefined,
  'plain': false,
  'staticClass': 'list',
  'classBinding': 'bindCls',
  'children': [{
    'type': 1,
    'tag': 'li',
    'attrsList': [{
      'name': '@click',
      'value': 'clickItem(index)'
    }],
    'attrsMap': {
      '@click': 'clickItem(index)',
      'v-for': '(item,index) in data'
     },
    'parent': // ul ast element
    'plain': false,
    'events': {
      'click': {
        'value': 'clickItem(index)'
      }
    },
    'hasBindings': true,
    'for': 'data',
    'alias': 'item',
    'iterator1': 'index',
    'children': [
      'type': 2,
      'expression': '_s(item)+":"+_s(index)'
      'text': '{{item}}:{{index}}',
      'tokens': [
        {'@binding':'item'},
        ':',
        {'@binding':'index'}
      ]
    ]
  }]
}
```



### 如何生成AST？

主要通过正则匹配来解析模板，流程图如下：

![](https://ustbhuangyi.github.io/vue-analysis/assets/parse.png)

![](https://user-gold-cdn.xitu.io/2019/5/26/16af44ef4c97f6ef?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## optimize（优化AST）

### 为什么要优化AST？

因为Vue是数据驱动、响应式的，但是我们的模板并不是所有数据都是响应式的，也有很多数据是首次渲染后就永远不会变化的，那么这部分数据生成的 DOM 也不会变化，我们可以在 `patch` 的过程跳过对他们的比对。

### 如何进行优化？

`optimize` 的过程，就是深度遍历这个 AST 树，去检测它的每一颗子树是不是静态节点，如果是静态节点则它们生成 DOM 永远不需要改变，这对运行时对模板的更新起到极大的优化作用。

我们通过 `optimize` 我们把整个 AST 树中的每一个 AST 元素节点标记了 `static` 和 `staticRoot`，它会影响我们接下来执行代码生成的过程。



## generate

这里就是把经过优化的AST转化为render函数。