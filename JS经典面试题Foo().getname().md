# JS经典面试题Foo().getname()

题目如下：

```javascript
function Foo() {
    getName = function () {
        console.log(1);
    };
    return this;
};
Foo.getName = function () {
    console.log(2);
};
Foo.prototype.getName = function () {
    console.log(3);
};
var getName = function () {
    console.log(4);
};
function getName() {
    console.log(5);
};

Foo.getName(); 
getName(); 
Foo().getName(); 
getName(); 
new Foo.getName();
new Foo().getName();
new new Foo().getName();
```



输出看结果：

```javascript
Foo.getName(); //2
getName(); //4
Foo().getName(); //1
getName(); //1
new Foo.getName(); //2
new Foo().getName(); //3
new new Foo().getName(); //3
```



其中最难理解的是有关new的最后三道题(5、6、7题)，因此先从这三道题说起。

这三道题考的是运算符优先级，先来看三道题是如何执行的。

#### new Foo.getName();

`new (Foo.getName)()`:先获取`function Foo.getName`，再`new`这个`function`，因此输出2



#### new Foo().getName(); 

先`new Foo()`得到一个实例对象，然后执行实例中的`getName()`方法，因为实例中没有这个方法，但原型上有，因此执行的是`Foo.prototype.getName`，输出3

**与上一题相比较的话就产生一个疑惑，为什么这个会先`new`，而上一题是后`new`，这个问题待会分析。**



#### new new Foo().getName();

如果看不懂上面两题，这一题也肯定看不懂，因此对这题先做了解就好。

首先执行中间的`new Foo()`得到一个实例对象(假设为foo),再次强调`foo`是一个对象,此时变成了`new foo.getName()`

接着执行`foo.getName`获取到foo对象中的`getName`属性，实则就是`Foo.prototype.getName`

最后执行`new`，相当于`new Foo.prototype.getName`，因此输出3



#### 解惑：为什么第5题后new，第6题先new

这与javascript运算符优先级有关，下面简单介绍一下运算符的优先级。

详细的运算符优先级表格在这里：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Operator_Precedence。

在这个题目中我们只需要关注题目出现过的运算符，有三个：函数调用`()`、成员访问`.`和`new`关键字，巧的是这三个运算符的优先级相同(优先级为20)。

这里要注意一个细节，`new`(带参数列表)的优先级为20，`new`无参数列表的优先级为19，这也是判断第5和6题的关键。

什么是带参数列表？简单的说就是`new Foo()`，而无参数列表是`new Foo`，没有括号，综合上面的说法，`new Foo`比函数调用`()`、成员访问`.`的优先级低。

现在我们可以彻底理解第5题new Foo.getName()的执行顺序，为什么是后new？

我们可以分析`new Foo.getName()`可能的执行顺序：

1. 先`new Foo`返回的是一个`foo`实例对象，再执行`foo.getName()`,最后输出3。
   - 这里第一步的优先级为19，第二步为20
2. 与正确答案一样，先`Foo.getName`再`new`，最后输出2。
   - 这里第一步的优先级为20，第二步的优先级为20.

比较两种顺序的第一步，答案很明显，下面的执行顺序优先级更高。



#### 再来分析第6题，为什么先new？

`new Foo().getName() `

可能的执行顺序：

1. 先`new Foo()`得到实例对象`foo`，再执行`foo.getName()`，输出3
   - 第一步的优先级为20，第二步为20
2. 先执行`Foo().getName`得到一个`Foo`的静态方法，再`new`这个静态方法，最后输出2
   - 第一步的优先级是20，第二部的优先级也是20

这是什么情况？既然大家的第一步优先级都是20，那为什么第一种是正确的？

这里要考虑运算符的关联性，详细可以看上面发的MDN链接。简单地说，一个运算式可能会从左到右执行，也可能从右到左执行，取决于运算符的关联性。而在这个例子中，`new`是没有关联性的，函数调用`()`、成员访问`.`的关联性是从左到右，这意味优先级相同时，`new Foo().getName() `应该从左到右执行。左边的第一个运算符是`new`，当然是先执行`new`。



到目前为止，我们已经搞懂了第5题和第6题，那么最后一题相信大家思考一下都能得到答案。，而1-4题也是比较简单的题目。



### 总结

解题的思路是运算符的优先级，以及优先级相同时考虑关联性。