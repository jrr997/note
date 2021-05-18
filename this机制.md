# this机制





参考文章：http://www.ruanyifeng.com/blog/2018/06/javascript-this.html



## 什么时候会用到this？

1. 函数中，包括构造函数
2. call、apply、bind等方法



## this指向

首先我们来体验一下this

 ```javascript
 var obj = {
   foo: function () { console.log(this.bar) },
   bar: 1
 };
 
 var foo = obj.foo;
 var bar = 2;
 
 obj.foo() // 1
 foo() // 2
 ```

有前端基础的人都能知道上面这段代码为什么会输出不同的结果，很多教科书会说this指向**函数运行时的环境**，`obj.foo()`中`this`指向`obj`，而`foo()`中`this`指向`window`，因此输出的结果不一样。

那么什么是**函数运行时的环境**呢？



## 函数的运行环境

首先我们要知道函数的方式：

1. 作为函数被调用，此时this指向window(非严格模式)
2. 作为对象的方法被调用，this指向离函数最近的对象
3. call、apply、bind，this指向传入的对象
4. new：this指向一个新建的空对象(这里要自行了解new的执行机制)

这里忽略了箭头函数，箭头函数的this指向很简单，就是指向箭头函数所在的环境中的this，如果箭头函数所在的环境中没有this(常见的情况是箭头函数中又有箭头函数)，则继续向外层环境找。



知道了以上这些，其实就已经知道了this指向的机制。但仅靠这些知识并不足够我们判断this指向，原因在Javascript的存储机制。









回顾这个例子：

```javascript
var obj = {
  foo: function () { console.log(this.bar) },
  bar: 1
};

var foo = obj.foo;
var bar = 2;

obj.foo() // 1
foo() // 2
```

我们声明了全局变量`foo = obj.foo`，按道理`foo()`就等同于`obj.foo()`，而问题就出在这里，两者并不相等，原因是javascript的存储机制。



## 存储问题

javascript的存储机制：

在javascript中变量存储的数据类型分为简单数据类型(number,string,boolean等)和引用数据类型(array、object、function)

对于简单类型，变量直接存储值；对于引用数据类型，变量存储的是数据的引用(内存地址)



在这个例子中，我们声明了3个全局变量`obj`、`foo`和`bar`，前两个是引用类型，而`bar`则存储的是值

`obj`的存储结构如下：

![](https://www.wangbase.com/blogimg/asset/201806/bg2018061802.png)

其中变量`obj`存储了`foo`属性，而`foo`属性对应的值是一个函数(引用类型)，因此`foo`存储了这个函数的内存地址。

而全局变量`foo`保存者和`obj.foo`相同的内存地址，可以说`foo === obj.foo`，实质上保存的是相同的函数，此时全局变量的`foo`和全局变量`obj`是没有关系的，因此`foo()`相当于在全局中直接执行了一个function。

而`obj.foo()`是在对象`obj`环境中执行这个function,因此`this`指向`obj`。



了解了这一点，以后遇到类似的情况，我们可以从储存方面分析this的指向，当然我觉得自己的表达其实不够好，可以参考这篇文章：[阮一峰javascript的this原理](http://www.ruanyifeng.com/blog/2018/06/javascript-this.html)