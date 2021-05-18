# HTTP1.1新特性

## HTTP1.1特性：

- 默认长连接：新增Connection字段，可以设置keep-alive值保持连接不断开。
- 管道化：可以在发送一个请求后继续发送第二个请求，不必等第一个请求的响应。
- 缓存处理：新增字段cache-control，ETag。



缓存：

- 强缓存：expires(HTTP1.0)、cache-control(HTTP1.1)

  在cache-control中，

  有max-age表示过期时间是时间戳。

  no-store表示不使用缓存，

  no-cache表示不使用协商缓存

- 协商缓存：Last-Modified(HTTP1.0)和If-modified-since，Etag(HTTP1.1)和If-none-match。



注意：cache-control优先级比expires高。