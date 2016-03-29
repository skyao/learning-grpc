Java生成代码
==========

> 注： 内容来自官网资料 [Java Generated Code](https://developers.google.com/protocol-buffers/docs/reference/java-generated)

这个页面准确描述 protocol buffer 编译器为任何给定协议定义生成的java代码。proto2和proto3生成的代码之间的任何不同都将被高亮 - 注意在这份文档中描述的是这些生成代码的不同，而不是基本的消息类/接口，后者在两个版本中是相同的。在阅读这份文档之前你应该先阅读 proto2语言指南 和/或 proto3语言指南。

> 说明：这个文档的英文原文是放在一个页面中，有点长，阅读起来比较累，因此我翻译时按照章节拆分为多个页面。

章节列表：

* [编译器调用](proto3/reference/java/compiler_invocation.md)
* [包](proto3/reference/java/packages.md)
* [消息](proto3/reference/java/messages.md)
* [字段](proto3/reference/java/fileds.md)
* [Any](proto3/reference/java/any.md)
* [Oneof](proto3/reference/java/oneof.md)
* [枚举](proto3/reference/java/enumerations.md)
* [扩展](proto3/reference/java/extensions.md)
* 服务(忽略)
* 插件插入点(忽略)

> 注： 服务和插件插入点这两节在这里跳过不做详细翻译，因为这两节讲述的内容和grpc无关，而我主要关注grpc的使用，在没有grpc的情况下如何生成通用的service和如何使用插件对我来说意义不大。