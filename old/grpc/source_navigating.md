源码导航
=======

> 注：内容翻译自grpc-java首页的 [Navigating Around the Source](https://github.com/grpc/grpc-java)。

从高水平上看，类库有三个不同的层: __Stub/桩__, __Channel/通道__  & __Transport/传输__.

# Stub

Stub层暴露给大多数开发者，并提供类型安全的绑定到正在适应（adapting）的数据模型/IDL/接口。gRPC带有一个protocol-buffer编译器的 [插件](https://github.com/google/grpc-java/blob/master/compiler)用来从`.proto` 文件中生成Stub接口。当然，到其他数据模型/IDL的绑定应该是容易添加并欢迎的。

### 关键接口

[Stream Observer](https://github.com/google/grpc-java/blob/master/stub/src/main/java/io/grpc/stub/StreamObserver.java)

# Channel

Channel层是传输处理之上的抽象，适合拦截器/装饰器，并比Stub层暴露更多行为给应用。它想让应用框架可以简单的使用这个层来定位横切关注点（address cross-cutting concerns）如日志，监控，认证等。流程控制也在这个层上暴露，容许更多复杂的应用来直接使用它交互。

### Common

* [元数据 - headers & trailers](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/Metadata.java)
* [状态 - 错误码命名空间 & 处理](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/Status.java)

### Client

* [Channel - 客户端绑定](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/Channel.java)
* [Client Call](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/ClientCall.java)
* [Client 拦截器](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/ClientInterceptor.java)

### Server

* [Server call handler - analog to Channel on server](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/ServerCallHandler.java)
* [Server Call](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/ServerCall.java)

# Transport

Transport层承担在线上放置和获取字节的繁重工作。它的接口被抽象到恰好刚刚够容许插入不同的实现。Transport被建模为`Stream`工厂。server stream和client stream之间在接口上的存在差别以整理他们在取消和错误报告上的不同语义。

注意transport层的API被视为gRPC的内部细节，并比在package `io.grpc`下的core API有更弱的API保证。

gRPC带有三个Transport实现：

1. [基于Netty](https://github.com/google/grpc-java/blob/master/netty) 的transport是主要的transport实现，基于[Netty](http://netty.io). 可同时用于客户端和服务器端。
2. [基于OkHttp](https://github.com/google/grpc-java/blob/master/okhttp) 的transport是轻量级的transport，基于[OkHttp](http://square.github.io/okhttp/). 主要用于Android并只作为客户端。
3. [inProcess](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/inprocess) transport 是当服务器和客户端在同一个进程内使用使用。用于测试。

### Common

* [Stream](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/internal/Stream.java)
* [Stream Listener](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/internal/StreamListener.java)

### Client

* [Client Stream](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/internal/ClientStream.java)
* [Client Stream Listener](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/internal/ClientStreamListener.java)

### Server

* [Server Stream](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/internal/ServerStream.java)
* [Server Stream Listener](https://github.com/google/grpc-java/blob/master/core/src/main/java/io/grpc/internal/ServerStreamListener.java)

