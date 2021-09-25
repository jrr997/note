# HTTP2

### 前言

**Q：为什么要HTTP2？**

**A：因为HTTP1.x有缺陷。**

- 连接无法复用，一个TCP连接对应一个http请求。每次请求都要新建一个TCP连接，而TCP连接要经过三次握手和慢启动，效率较低。

  - HTTP1.0中，每个请求都要重新建立一个连接。
  
- 队头阻塞
  
  - HTTP1.1加入了keep-alive，虽然可以在同一个TCP连接中发送多个HTTP请求，但是在同一时间、同一域名下的请求有一定数量限制，超过限制数目的请求会被阻塞。这意味着某个请求的超时会导致后面的请求阻塞(线头阻塞)，HTTP1.1发送请求是单线程的。
  
- 头部开销大

  在HTTP1.x，每次请求都会在header中携带大量的信息，而这些信息有很多都是重复的，这方面有很大的优化空间(HTTP2.0就优化了这方面)。

**HTTP2旨在在HTTP1.x的基础上，改善web性能，可以认为是在HTTP1.x的基础上进行扩展。**



那么HTTP2做了什么工作呢？

**一句话概括就是HTTP2的新特性：二进制分帧、多路复用、Header压缩、服务端推送。**



### 1.二进制分帧

这里有两个重要概念：**二进制**和**分帧**

#### 二进制

Q：为什么是二进制？

A：HTTP2采用二进制传输，而HTTP1.x传输的是文本，由于二进制只有0和1，因此数据解析起来更高效。

#### 分帧

Q1：什么是分帧？

A1：这里要先了解几个概念：

**流**：流是连接中的一个虚拟信道，可以承载双向的消息；每个流都有一个唯一的整数标识符（1、2…N）；

**消息**：相当于HTTP请求，由一个或多个帧组成;

**帧**：分为**Header帧**和**Data帧**，**Header帧**保存的是HTTP请求头部的信息，**Data帧**保存的是数据信息，包括请求参数和响应结果。帧是最小的传输单位，帧的首部会标识出当前帧所属的流，因为HTTP2把一个请求分为多个帧传输，那么如何区分某个帧是哪个请求呢？这个标志的流用以区分不同的HTTP请求。

![](https://image.fundebug.com/2019-03-06-2.png)

总结一下：消息(初步理解为HTTP请求)在流中传输，而消息又拆分成多个帧用来传输。当然一个流可以传输多个信息，只要在帧的首部进行区分就行。

Q2：为什么要分帧？

A2：分帧主要用在多路复用中，解决HTTP1.x中数据传输阻塞的问题，下面有更具体的描述。



### 2.多路复用

- **双向传输**：在同一个TCP连接中可以传输任意数量的双向数据流，这就突破了HTTP1.1对HTTP请求数量的限制。
- **一个TCP连接**：同一域名下数据的传输在一个TCP连接中完成。在HTTP1.x中，一个网页可能会同时与服务器建立多个TCP连接来加快资源获取的速度，而在HTTP2不需要这么做。
- **帧乱序传输**：注意HTTP2的数据传输时双向的，这意味着请求和响应可以在同一个TCP连接中同时传输。，数据可以乱序传输，因为每个数据帧都会记录了自己属于哪个请求以及自己的顺序，只要在数据传输的终点根据帧数据的提供信息把帧拼接起来，即可获取一个完整的请求。这么做的好处是解决了HTTP1.x中的阻塞问题。
- **并行传输**：可以并行传输多个HTTP请求（多流）。在HTTP1.1时虽然有长连接和管道化，但长连接的同一时刻只有一个请求被发送，要实现并行请求，得多个TCP连接。



### 3.优先值

HTTP2中每个请求都可以携带一个 31bit 的优先值，0 表示最高优先级， 数值越大优先级越低。有了这个优先值，客户端和服务器就可以在处理不同的流时采取不同的策略，以最优的方式发送流、消息和帧。



### 4.头部压缩

文章的简介中提到过HTTP1.x中的请求头部开销大，有很大的优化空间，HTTP2就是用压缩策略(HPACK)来进行优化。

下面这张截图，取自 Google 的性能专家 Ilya Grigorik 在 Velocity 2015 • SC 会议中分享的「[HTTP/2 is here, let's optimize!](http://velocityconf.com/devops-web-performance-2015/public/schedule/detail/42385)」，非常直观地描述了 HTTP/2 中头部压缩的原理：

![hpack-header-compression](https://st.imququ.com/i/webp/static/uploads/2015/10/hpack-header-compression.png.webp)

通俗地讲：

1. 服务端和客户端在建立连接后共同维护一份静态字典(Static table)，字典中包含常见的头部名称，以及特别常见的头部名称与值的组合；

   那么静态字典有什么用呢？

   我们可以看到字典中每个键值对都有索引值，如果我们发送的HTTP请求的头部信息和静态字典中的某一项完全匹配，那么我们用索引值就能代表某个请求头部信息，这样就大大缩小了头部信息的大小。

2. 维护一份相同的动态字典（Dynamic Table），可以动态地添加内容；

   如果请求的某一个头部信息在静态字典中匹配不到，那么我们需要把这个头部信息放进动态字典中并更新字典，那么以后我们就能继续用索引值来表示这个头部信息。

3. 字典中的数据支持哈夫曼编码。

   简单介绍一下哈夫曼编码：

   哈夫曼编码(Huffman Coding)是一种编码方式，以哈夫曼树—即最优二叉树，带权路径长度最小的二叉树，经常应用于数据压缩。在计算机信息处理中，“哈夫曼编码”是一种一致性编码法（又称"熵编码法"），用于数据的无损耗压缩。这一术语是指使用一张特殊的编码表将源字符（例如某文件中的一个符号）进行编码。这张编码表的特殊之处在于，它是根据每一个源字符出现的估算概率而建立起来的（出现概率高的字符使用较短的编码，反之出现概率低的则使用较长的编码，这便使编码之后的字符串的平均期望长度降低，从而达到无损压缩数据的目的）。这种方法是由David.A.Huffman发展起来的。例如，在英文中，e的出现概率很高，而z的出现概率则最低。当利用哈夫曼编码对一篇英文进行压缩时，e极有可能用一个位(bit)来表示，而z则可能花去 25个位（不是26）。用普通的表示方法时，每个英文字母均占用一个字节（byte），即8个位。二者相比，e使用了一般编码的1/8的长度，z则使用了 3倍多。倘若我们能实现对于英文中各个字母出现概率的较准确的估算，就可以大幅度提高无损压缩的比例。

   

### 5.服务端推送

当客户端向服务端请求一个HTML文件时，客户端很可能需要继续请求相关的CSS、JS文件。在HTTP2之前，客户端会在解析HTML时发现依赖的CSS、JS、图片等资源，然后向指定的地址来发送请求。

我们能从中发现问题：一是需要多轮HTTP请求，二是收到样式文件之前，网页都会显示一片空白，这个阶段一旦超过2秒，用户体验就会非常不好。



在HTTP2之前的解决方案：

一种解决办法就是把外部资源合并在网页文件里面，减少 HTTP 请求。比如，把样式表的内容写在`<style>`标签之中，把图片改成 Base64 编码的 [Data URL](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/Data_URIs)。

另一种方法就是资源的[预加载](https://w3c.github.io/preload/)（preload）。网页预先告诉浏览器，立即下载某些资源。



这两种方法都有缺点。第一种方法虽然减少了 HTTP 请求，但是把不同类型的代码合并在一个文件里，违反了分工原则。第二种方法只是提前了下载时间，并没有减少 HTTP 请求。





HTTP2的服务端推送能解决这个问题。

#### 什么是服务端推送？

服务器推送（server push）指的是，还没有收到浏览器的请求，服务器就把各种资源推送给浏览器。

比如，浏览器只请求了`index.html`，但是服务器把`index.html`、`style.css`、`example.png`全部发送给浏览器。这样的话，只需要一轮 HTTP 通信，浏览器就得到了全部资源，提高了性能。

![](https://image.fundebug.com/2019-03-06-6.png)

#### 服务端推送的过程？

PUSH_PROMISE 101

所有服务器推送数据流都由 `PUSH_PROMISE` 帧发起，表明了服务器向客户端推送所述资源的意图，并且需要先于请求推送资源的响应数据传输。 这种传输顺序非常重要: 客户端需要了解服务器打算推送哪些资源，以免为这些资源创建重复请求。 满足此要求的最简单策略是先于父响应（即，`DATA` 帧）发送所有 `PUSH_PROMISE` 帧，其中包含所承诺资源的 HTTP 标头。

在客户端接收到 `PUSH_PROMISE` 帧后，它可以根据自身情况选择拒绝数据流（通过 `RST_STREAM` 帧）。 （例如，如果资源已经位于缓存中，便可能会发生这种情况。） 这是一个相对于 HTTP/1.x 的重要提升。 相比之下，使用资源内联（一种受欢迎的 HTTP/1.x“优化”）等同于“强制推送”: 客户端无法选择拒绝、取消或单独处理内联的资源。

使用 HTTP/2，客户端仍然完全掌控服务器推送的使用方式。 客户端可以限制并行推送的数据流数量；调整初始的流控制窗口以控制在数据流首次打开时推送的数据量；或完全停用服务器推送。 这些优先级在 HTTP/2 连接开始时通过 `SETTINGS` 帧传输，可能随时更新。

推送的每个资源都是一个数据流，与内嵌资源不同，客户端可以对推送的资源逐一复用、设定优先级和处理。 浏览器强制执行的唯一安全限制是，推送的资源必须符合原点相同这一政策: 服务器对所提供内容必须具有权威性。

以上这段话其实说得不够深入。



Server push 的原理很简单，本质上就是**先替你请求再告诉你**。

假设服务端接收到客户端对 HTML 文件的请求，决定用 server push 推送一个样式表文件。那么，服务端会构造一个请求，包括请求方法和请求头，填充到一个 [PUSH_PROMISE](https://link.zhihu.com/?target=https%3A//http2.github.io/http2-spec/index.html%23PUSH_PROMISE) 帧里发送给客户端，来告知客户端它已经代劳发了这个请求。客户端可以根据 PUSH_PROMISE 帧里提供的 Promised Stream Id 来读推过去的响应。

![img](https://pic2.zhimg.com/80/v2-3b337b0e6f342b6fd0a31b8fe18eee15_1440w.png)

当客户端收到这个 PUSH_PROMISE 帧的时候，它就知道服务端将要推送一个样式表文件回来。如果此时客户端需要请求这个样式表文件，即便服务端还没推完，它也不会往服务端发送对样式表文件的请求。

这里需要注意的是避免**竞争**。在上面的例子中，必须先发送 PUSH_PROMISE，再发送 HTML 的内容。这是因为 HTML 中存在对样式表文件的引用，一旦客户端发现了这个引用却还没收到 PUSH_PROMISE，它就会发起请求。这会引起 PUSH_PROMISE 和对样式表文件的请求之间的竞争，从而 server push 有一定的几率失败。

另一种竞争是不可避免的。如果客户端认为它不需要某个即将被推过来的资源（比如这个资源还在缓存的有效期内），那么它会 reset 掉相应的流。但是即便如此，服务端在收到 RST_STREAM 帧的时候，很有可能已经推了一部分数据了。这种服务端开始推送数据和 RST_STREAM 帧之间的竞争是难以避免的（这是 feature 而不是 BUG）。



#### 服务端推送的实现

**标识依赖资源**

W3C候选推荐标准（https://www.w3.org/TR/preload/）建议了依赖资源的两种做法：文件内<link>标签和HTTP头部携带, 表示该资源后续会被使用, 可以预请求, 关键字preload修饰这个资源, 写法如下：

**a) 静态Link标签法:**

<link rel="preload" href="push.css" as="style">



**b) HTTP头表示法：**

Link: <push.css>; rel=preload; as=style



其中rel表明了资源</push.css>是预加载的，as表明了资源的文件类型。另外，link还可以用nopush修饰，表示浏览器可能已经有该资源缓存，指示有推送能力的服务端不主动推送资源，只有当浏览器先检查到没有缓存，才去指示服务端推送资源，nopush格式写成：

Link: </app/script.js>; rel=preload; as=script;nopush。



总结一下：

客户端向服务端请求一个html文件，服务端进行推送，并把相关的资源(或者资源链接)推送给客户端(通过HTTP请求头的LINK或者HTML中的preload)。

客户端知道了服务端要推送相关资源，可以选择接收和不接收。如果客户端不接收，可以通过 RST_STREAM 帧告诉服务端，服务端会停止推送。

服务端为了避免重复推送，可以在请求头的LINK中加上nopush，服务端看到nopush后会检查自己是否有相关资源，然后选择要不要服务端推送。





参考文章：

1.[HTTP/2 头部压缩技术介绍](https://imququ.com/post/header-compression-in-http2.html)

2.[一文读懂 HTTP/2 及 HTTP/3 特性](https://blog.fundebug.com/2019/03/07/understand-http2-and-http3/)

3.[哈夫曼编码的作用

4.[HTTP/2之服务器推送(Server Push)最佳实践](http://www.duorenwei.com/news/1369.html)

5.[HTTP/2 简介](https://developers.google.com/web/fundamentals/performance/http2?hl=zh-cn#push_promise_101)

 