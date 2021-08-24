---
weight: 2
bookFlatSection: true
title: "初识gRPC"
---

在正式写`grpc-shop`之前，我们需要先花几篇文章介绍一些基础概念。 在介绍`gRPC`之前我们先了解下什么是`RPC`。


### **什么是rpc?**

`RPC`是`Remote Procedure Call`的简称，中文叫远程过程调用。

说的白话一点，可以这么理解：现在有两台服务器A和B。部署在A服务器上的应用，想调用部署在B服务器上的另一个应用提供的方法，由于不在一个内存空间，不能直接调用，需要通过网络来达到调用的效果。

现在我们在A服务的一个本地方法中封装调用B的逻辑，然后只需要在本地使用这个方法，就达到了调用B的效果。

对使用者来说，屏蔽了细节。你只需要知道调用这个方法返回的结果，而无需关注底层逻辑。

那，从封装的那个方法角度来看，调用B之前我们需要知道什么？

当然是一些约定啊。

- 调用的语义，也可以理解为接口规范。(比如`RESTful`)
- 网络传输协议 (比如`HTTP`)
- 数据序列化反序列化规范(比如`JSON`)。
  
有了这些约定，我就知道如何给你发数据，发什么样的数据，你返回给我的又是什么样的数据。

![rpc](https://cdn.syst.top/rpc.jpeg)

从上图中可以看出，`RPC`是一种客户端-服务端（`Client/Server`）模式。

从某种角度来看，所有本身应用程序之外的调用都可以归类为`RPC`。无论是微服务、第三方`HTTP`接口，还是读写数据库中间件`Mysql`、`Redis`。



### **HTTP 和 RPC 有什么区别？**

我之前也问个这个问题。

首先这个问题本身不太严谨。

` HTTP`只是一个通信协议，工作在`OSI`第七层。

 而`RPC`是一个完整的远程调用方案。它包含了:接口规范、传输协议、数据序列化反序列化规范。

这样看，`RPC`和 `HTTP`的关系只可能是包含关系。为什么是可能？因为`RPC`传输协议那块我可以不基于`HTTP`呀。

所以这个问题应该改成:基于`HTTP`的远程调用方案 (如:`HTTP`+`RESTful`+`JSON`) 和直接使用`RPC`远程调用方案有什么区别？



#### **RPC 和 gRPC 有什么关系？**

`gRPC`是由 `google`开发的一个高性能、通用的开源`RPC`框架，主要面向移动应用开发且基于`HTTP/2`协议标准而设计，同时支持大多数流行的编程语言。

`gRPC`基于 `HTTP/2`协议传输。而`HTTP/2`相比`HTTP1.x`:

##### 用于数据传输的二进制分帧

`HTTP/2`采用二进制格式传输协议，而非`HTTP/1.x`的文本格式。

![data](https://cdn.syst.top/data.jpeg)

##### 多路复用

`HTTP/2`支持通过同一个连接发送多个并发的请求。

而`HTTP/1.x`虽然通过`pipeline`也能并发请求，但多个请求之间的响应依然会被阻塞。 

![http1.0-pipeline](https://cdn.syst.top/pipelining.png)

##### 服务端推送

服务端推送是一种在客户端请求之前发送数据的机制。在`HTTP/2`中，服务器可以对客户端的一个请求发送多个响应。而不像`HTTP/1.X`一样，只能通过客户端发起`request`,服务端才产生对应的`response`。



##### 减少网络流量的头部压缩。

`HTTP/2`对消息头进行了压缩传输，能够节省消息头占用的网络流量。至于如何压缩的，可以查看这篇：[HPACK: Header Compression for HTTP/2](https://httpwg.org/specs/rfc7541.html)

同时`gRPC`使用`Protocol Buffers`作为序列化协议。关于`Protocol Buffers`。官网有一句介绍：

> Protocol buffers are Google’s language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler.

它是一种与语言、平台无关 、可扩展的序列化结构数据。它的定位类似于`JSON`、`XML`，但是比他们更小、更快、更简单。更多关于`Protocol Buffers`介绍，我下一篇再写。





### **gRPC 是如何进行远程调用的?**



官网有一张图:

![图片](https://cdn.syst.top/grpc.png)



从上图和文档中可以看出，用`gRPC`来进行远程调用服务，客户端(`client`) 仅仅需要`gRPC Stub`(为啥叫存根?) ，通过`Proto Request`向`gRPC Server`发起服务调用，然后 `gRPC Server`通过`Proto Response(s)`将调用结果返回给调用的`client`。

至于上面这段逻辑`gRPC`里面做了啥，有哪些调用方式，介绍完`pb`再写。



### 总结

第一篇文章主要介绍了`RPC`是什么以及一些`grpc`的基础概念。





### 参考
- https://grpc.io/docs/what-is-grpc/introduction/
- https://httpwg.org/specs/rfc7541.html
- https://www.upyun.com/tech/article/227/1.html?utm_source=zhihu&utm_medium=referral&utm_campaign=202831825&utm_term=http2
- https://www.zhihu.com/question/34074946

