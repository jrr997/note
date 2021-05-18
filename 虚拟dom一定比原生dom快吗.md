# 虚拟DOM一定比原生DOM快吗



## 一、虚拟DOM Vs 原生DOM的渲染性能

原生操作DOM的步骤：

1. render html string：渲染html字符串
2. innerHtml插入DOM

虚拟Dom：

1. render：生成虚拟DOM(Js对象)
2. diff
3. 更新DOM



首先Virtual DOM render + diff 显然比渲染 html 字符串要慢，但是原生操作DOM要创建所有真实DOM，而虚拟DOM只需要更新发生改变的数据，创建对应的真实Dom。

虚拟DOM的优势在于只更新有变化的DOM，而原生需要更新所有DOM，无论DOM有没发生变化。



## 二、框架封装操作 Vs 原生Dom操作



虚拟DOM出现在MVVM框架中，像Vue、React、Angular，原生DOM主要用innerHTML来操作。

1. 框架中的DOM是对原生DOM的封装，封装的意义在于掩盖底层DOM操作，让开发者能够以声明的方式来操作DOM，从而使代码更容易维护。
2. 框架对DOM进行过优化，但是没有任何框架的DOM比纯手动优化的DOM快。因为框架中操作DOM需要应对上层API可能产生的操作，它的实现必须是普适的。这是性能和可维护性的取舍。



## 三、性能比较要看场合

在比较性能的时候，要分清楚初始渲染、小数据更新和大数据更新。

例如在初始渲染时原生操作需要创建所有的DOM，而框架需要先创建虚拟DOM，再根据虚拟DOM创建真实DOM，这无疑是原生操作DOM快。





参考文章：https://www.zhihu.com/question/31809713