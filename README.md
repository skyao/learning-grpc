Grpc学习笔记和资料翻译
===========

# 介绍

grpc是google最新发布(2015年2月底)的开源rpc框架, 声称是"一个高性能，开源，将移动和HTTP/2放在首位的通用的RPC框架.". 技术栈非常的新, 基于HTTP/2, netty5.0, proto3, 拥有非常丰富而实用的特性, 堪称新一代RPC框架的典型.

个人对grpc非常感兴趣, 正在努力学习中. 这里的笔记用于记录google grpc学习过程的各种信息和心得, 有些重要资料会顺便做个翻译(下面章节内容中有标注)。

> 注: **grpc目前已经进入beta阶段**, 已经可以考虑在正式产品中使用了. 当然，期待正式版尽快发布!

# 章节内容

现在已有的内容:

* [gPRC 介绍](introduction/index.md)
    * [gRPC 信息](introduction/information.md)
* [Protocol Buffer](proto3/index.md)
    * [开发指南](proto3/guide/index.md)
        * [概述(翻译)](proto3/guide/overview.md)
        * [语言指南(翻译)](proto3/guide/language_guide.md)
        * [风格指南(翻译)](proto3/guide/style_guide.md)
        * [编码(翻译)](proto3/guide/encoding.md)
        * [技巧(翻译)](proto3/guide/techniques.md)
	* [API参考文档](proto3/reference/index.md)
		* [Java Generated Code](proto3/reference/java/index.md)
* [gPRC文档]()
    * [官网开发指南](grpc/grpc.md)
    	* [概述(笔记)](grpc/overview.md)
    	* [gRPC概念(笔记)](grpc/grpc_concepts.md)
* [实战](action/index.md)
	* [集成Spring Boot](action/springboot/springboot.md)
	* [文档生成](action/documentation/index.md)

内容陆续添加中......

> 注: 如果你看到的是github的源代码, 请点击 [这里](http://skyao.github.io/leaning-grpc/) 查看html内容.