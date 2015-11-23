gRPC 动机和设计原则
================

注: 官网文档[gRPC Motivation and Design Principles](http://www.grpc.io/posts/principles/)的读书笔记.

# 动机

google使用名为Stubby的通用RPC基础设施来连接数量巨大的微服务.

但是Stubby不是基于标准而且和内部设施耦合太紧以至于不能对外发布. 随着SPDY, HTTP/2 和 QUIC的问世, 很多同样的特性出现在标准中, 而且还有一些Stubby不支持的特性.

很明显需要重新改造Stubby, 并扩展它的使用到移动, IoT和云.

# 原则和需求

- Services not Objects, Messages not References

    "服务而不是对象, 消息而不是引用".

    提倡在系统间进行粗颗粒消息交换的微服务设计哲学, 避开分布式对象的陷阱和忽略网络的谬见.










