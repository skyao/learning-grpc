Grpc学习笔记
===========

> 注: 如果你看到的是github的源代码, 请点击 [这里](http://skyao.github.io/leaning-grpc/) 查看html内容.

# 介绍

grpc是google最新发布(2015年2月底)的开源rpc框架, 声称是"一个高性能，开源，将移动和HTTP/2放在首位的通用的RPC框架.". 技术栈非常的新, 基于HTTP/2, netty4.1, proto3, 拥有非常丰富而实用的特性, 堪称新一代RPC框架的典范.

这里记录google grpc学习和使用过程的收集到的各种信息和心得, 另外针对性的翻译了部分特别有价值的英文资料(下面章节列表中有标注为"翻译")。

> 注: **grpc目前已经进入beta阶段**, 已经可以考虑在正式产品中使用. 当然，期待正式版尽快发布!

# 章节内容

* [gPRC 介绍](introduction/index.md)
    * [资料收集整理](introduction/information.md)
* [gPRC文档](grpc/index.md)
    * [gRPC官方文档(中文版)](grpc/official_docs.md)
    * [gRPC动机和设计原则](grpc/motivation.md)
* [Protocol Buffer 3](proto3/index.md)
* [gRPC源码学习](sourcecode/index.md)
	* [客户端代码](sourcecode/client/index.md)
	* [服务器端代码](sourcecode/server/index.md)
	* [通用代码](sourcecode/common/index.md)
* [实战](action/index.md)
	* [集成Spring Boot](action/springboot/springboot.md)
	* [文档生成](action/documentation/index.md)

内容陆续添加中......

