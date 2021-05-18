## 1.如何给node进程传递参数

```javascript
在命令行输入node index.js age=10 zhangsan
```

在全局变量`process.argv`获取参数

`argv`是argument vector的缩写。



## 2.node输出

`console.clear()`清空输出

`console.trace()`打印函数调用栈



## 3.全局对象

### 特殊的全局对象

- 不是真正的全局，只是模块中有
- 不能在命令行中使用
- 例如：`__dirname`、`__filename`等

### 常见全局对象

- `process`：提供Node进程中相关的信息

- `console`

- 定时器函数

  `setTimeout`、`setInterval`、`setImmediate`、`porcess.nextTick`

- `global`：类似于浏览器的`window`，但还是有区别。



## 4.模块化

- CommonJS、AMD、CMD是模块化的规范。

- Node中利用引用赋值实现CommonJS

  ![image-20210330162115577](C:\Users\Jrrr\AppData\Roaming\Typora\typora-user-images\image-20210330162115577.png)



### 导入

#### exports和module.exports的区别

在使用`exports`时，Node默认让`module.exports = exports`，而`exports = {}`，即两者指向同一个内存地址(同一个空对象)，这个默认的操作是发生在模块开始的，这意味着如果令`exports = 123`，`moduce.exports`还是指向一个空对象。

而其他模块中`require`拿到的其实是`module.exports`保存的内存地址。这意味如果我们使用`modele.xeports`进行导出，`exports`是不起作用的。

![image-20210330163707013](C:\Users\Jrrr\AppData\Roaming\Typora\typora-user-images\image-20210330163707013.png)

#### 那为什么要有exports和module.exports两个导出的API

因为CommonJS中规定要有`exports`中这个接口，因此`exports`是Node对CommonJS的妥协。



### 导出

1. `require`是一个函数，里面放一个字符串，包括内置模块、第三方模块和路径。

2. 路径规则：

- `require`路径包括`./  ../   /`
- 为什么不用加后缀名？
  - 因为node可以自动判断，如果没加后缀名且只能找到目录，那么会用目录中的index.js等用index命名的文件。

3.模块的加载过程

- `require`是同步加载，模块第一次加载时会被执行一次。

- 如果一个模块被多次被`reuqire`,模块中的代码只会被执行一次，因为有缓存。一个模块被加载（require）过后，这个模块的`module.loaded = true`，而没加载过或第一次加载时是`false`。
- 循环引用时，模块加载顺序为深度优先。

![image-20210330200346511](C:\Users\Jrrr\AppData\Roaming\Typora\typora-user-images\image-20210330200346511.png)



### ES模块化

异步、实时绑定、不是对象