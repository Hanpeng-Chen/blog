---
title: 一文彻底掌握HTTP缓存
urlname: http-cache
tags:
  - 前端
  - HTTP缓存
categories:
  - 前端
abbrlink: 17468
date: 2021-01-08 00:05:43
---

缓存，作为性能优化的最有效方法之一，在面试过程中经常被问到，作为一名开发人员，缓存是必须掌握的一块知识。

浏览器缓存机制有四个方面：Memory Cache、Service Worker Cache、HTTP Cache、Push Cache。

对于前端开发工程师来说，比较熟悉的就是HTTP缓存，这也是每一个前端工程师都要掌握的知识点，下面我们一起来学习HTTP缓存，争取通过这篇文章就彻底掌握HTTP缓存。


## HTTP缓存机制
HTTP缓存机制是根据HTTP报文的缓存标识进行的。HTTP缓存分为强缓存和协商缓存。优先级最高的是强缓存，在命中强缓存失败的情况下，才会走协商缓存。接下来我们一起来看下这两个缓存机制：

### 强缓存
强缓存是利用http头中的Expires和Cache-Control两个字段来控制的。强缓存中，当请求再次发出时，浏览器会根据其中的Expires和Cache-Control判断目标资源是否“命中”强缓存，如果命中则直接从缓存中获取资源，不会再与服务端发生通信。命中强缓存的情况下，返回的HTTP状态码为200。

#### Expires

过去我们一直使用Expires来实现强缓存：当服务器返回响应时，在Response Headers中将过期时间写入Expires字段。例如：

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/spring/20210107112445.png)

从上图我们可以看到Expires是一个时间戳，接下来如果我们试图再次向服务器请求资源，浏览器就会先对比本地时间和expires的时间戳，如果本地时间小于expires设定的过期时间，就直接从缓存中获取这个资源。

到这里聪明的你可能已经发现了下面这个问题：由于expires的时间戳是服务器定义的，而本地时间的取值来自客户端，因此expires的工作机制对于客户端时间和服务器时间的一致性要求极高，如果两者的时间存在时差，会带来意料之外的结果。

由于上面的原因，加上expires是HTTP1.0的产物，现在实现强缓存大多数是使用`Cache-Control`。


#### Cache-Control
Cache-Control是HTTP1.1提出的特性，为了弥补Expires缺陷提出的，提供了更精确细致的缓存功能。

Cache-Control包含的值很多：

- **public**：表明响应可以被任何对象（包括：发送请求的客户端、代理服务器等等）缓存。
- **private**：表明响应只能被客户端缓存。
- **no-cache**：跳过强缓存，直接进入协商缓存阶段。
- **no-store**：表示当前请求资源禁用缓存
- **max-age=<seconds>**：设置缓存存储的最大周期，超过这个时间缓存被认为过期（单位秒）
- **s-maxage=<seconds>**：覆盖max-age或者Expires头。如果s-maxage未过期，则向代理服务器请求其缓存内容。

> 这里需要注意的是：s-maxage仅在代理服务器中生效，客户端只需要考虑max-age。

下面我们来看个例子：

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/spring/20210107140339.png)

从上面的例子我们可以看到，HTTP响应报文中同时有Cache-Control和Expires两个字段，由于Cache-Control优先级较高，那么直接根据Cache-Control的值进行缓存，也就是说在315360000秒内重新发起该请求，会直接使用缓存结果，强制缓存生效。


在 HTTP1.1 标准试图将缓存相关配置收敛进 Cache-Control 这样的大背景下， max-age可以视作是对 expires 能力的补位/替换。在当下的前端实践里，我们普遍会倾向于使用max-age。但如果你的应用对向下兼容有强诉求，那么 expires 仍然是不可缺少的。



### 协商缓存
协商缓存，也称为对比缓存。协商缓存机制下，浏览器需要向服务器去询问缓存的相关信息，进而判断是重新发起请求、下载完整的响应，还是从本地获取缓存的资源。

如果服务端提示缓存资源未改动（Not Modified），资源会被重定向到浏览器缓存，这种情况下网络请求对应的状态码是 304（如下图）。

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/spring/20210107112113.png)

协商缓存依赖于服务端与浏览器之间的通信。

同样，协商缓存的标识也是在响应报文的HTTP头中和请求结果一起返回给浏览器的，控制协商缓存的字段分别有：Last-Modified和Etag，其中Etag的优先级比Last-Modified高。

#### Last-Modified & If-Modified-Since
Last-Modified（Response Header）和If-Modified-Since（Request Header）是一对报文头，属于HTTP1.0。

Last-Modified表示资源的最后修改时间，是一个时间戳，如果启用了协商缓存，它会在首次请求时随着Response Headers返回。
```
Last-Modified: Sat, 09 May 2020 09:33:56 GMT
```

If-Modified-Since是一个请求首部字段，并且只能用在GET或HEAD请求中。客户端再次请求服务器时，请求头会包含这个字段，后面跟着在缓存中获取的资源的最后修改时间。

```
If-Modified-Since: Sat, 09 May 2020 09:33:56 GMT
```

服务端收到请求发现此请求头中有If-Modified-Since字段，会与被请求资源的最后修改时间进行对比，如果一致则会返回304和响应报文头，浏览器从缓存中获取数据即可。从字面上看，就是说从某个时间节点开始看，是否被修改了，如果被修改了，就返回整个数据和200 OK，如果没有被修改，服务端只要返回响应头报文，304 Not Modified，Response Headers不会再添加Last-Modified字段。

使用Last-Modified是有一定缺陷的：
- 如果资源更新的速度是秒以下单位，那么该缓存是不能被使用的，因为 If-Modified-Since 只能检查到以秒为最小计量单位的时间差。
- 如果文件是通过服务器动态生成的，那么该方法的更新时间永远是生成的时间，尽管文件可能没有变化，所以起不到缓存的作用。
- 我们编辑了文件，但文件的内容没有改变。服务端并不清楚我们是否真正改变了文件，它仍然通过最后编辑时间进行判断。因此这个资源在再次被请求时，会被当做新资源，进而引发一次完整的响应——不该重新请求的时候，也会重新请求。

为了解决上面服务器没有正确感知文件变化的问题，Etag作为Last-Modified的补充出现了。


#### Etag & If-None-Match
Etag和If-None-Match是一对报文头，属于HTTP1.1。

Etag是一个响应首部字段，是根据实体内容生成的一段hash字符串，标识资源的状态，由服务端产生。
```
ETag: W/"324023994867772d0dd9fac01f1420bd"
```

If-None-Match是一个条件式的请求首部，如果请求资源时在请求首部加上这个字段，值为之前服务器返回的Etag，则当且仅当服务器上没有任务资源的Etag属性值与这个值相符，服务器才会返回带有请求资源实体的200响应，否正服务器会返回不带实体的304响应。
```
If-None-Match: W/"324023994867772d0dd9fac01f1420bd"
```

Etag 的生成过程需要服务器额外付出开销，会影响服务端的性能，这是它的弊端。因此启用 Etag 需要我们审时度势。正如我们刚刚所提到的：Etag 并不能替代 Last-Modified，它只能作为 Last-Modified 的补充和强化存在。 Etag 在感知文件变化上比 Last-Modified 更加准确，优先级也更高。当 Etag 和 Last-Modified 同时存在时，以 Etag 为准。


## 如何设置一个可靠的缓存规则

上面我们已经学完了HTTP缓存机制，是不是迫不及待想要实践一番？但是如何设置一个可靠的缓存规则，需要根据实际需求决定，绝大部分需求的缓存规则都可以根据Chrome官方提供的流程图来进行设置。

![](https://gitee.com/HanpengChen/blog-images/raw/master/blogImages/2021/spring/20210107155629.png)

我们一起来看上面这张流程图：

- 如果资源不可复用，直接为Cache-Control设置no-store，拒绝一切形式的缓存；
- 如果资源可复用，考虑是否每次都需要向服务器进行缓存确认，如果是，设置Cache-Control的值为no-cache；
- 如果不需要每次都向服务器确认，考虑资源是否可以被代理服务器缓存，根据其结果决定是设置为private还是public；
- 接下来考虑资源的过期时间，设置对应的max-age；
- 最后，配置协商缓存需要用到的Etag、Last-Modified等参数。

后续根据这个流程，我们就可以设计出可靠的缓存规则了。

看到这里，文章也即将结束，关于HTTP缓存你掌握了吗？希望大家看完这篇文章有所收获，以后面试碰到HTTP缓存的相关问题也能够从容应答。

## 参考资料
[Cache-Control MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)
[一文读懂前端缓存](https://juejin.cn/post/6844903747357769742#heading-6)
[前端性能优化原理与实践](https://juejin.cn/book/6844733750048210957/section/6844733750106931214)
