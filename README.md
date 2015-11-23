Grpc学习笔记
===========

# 介绍

grpc是google最新发布(2015年2月底)的开源rpc框架, 声称是"一个高性能，开源，将移动和HTTP/2放在首位的通用的RPC框架.". 技术栈非常的新, 基于HTTP/2, netty5.0, proto3, 拥有非常丰富而实用的特性, 堪称新一代RPC框架的典型.

注意: grpc目前还出于alpha阶段, 暂时还不适合用于

个人对grpc非常感兴趣, 正在努力学习中. 这里的笔记用于记录google grpc学习过程的各种信息和心得, 有些重要资料会顺便做个翻译(下面章节内容中有标注)。

# 章节内容

现在已有的内容:

* [google grpc 介绍](introduction/index.md)
    * [google grpc 信息](introduction/information.md)
* [Protocol Buffer]()
    * [官网开发指南]()
        * [概述(笔记)](proto3/overview.md)
        * [语言指南(翻译)](proto3/language_guide.md)
        * [风格指南(翻译)](proto3/style_guide.md)
        * [编码(翻译)](proto3/encoding.md)
        * [技巧(翻译)](proto3/techniques.md)
* [gPRC文档]()
    * [官网开发指南]()
    	* [概述(笔记)](grpc/overview.md)
    	* [gRPC概念(笔记)](grpc/grpc_concepts.md)

内容陆续添加中......

注: 如果你看到的是github的源代码, 请点击 [这里](http://skyao.github.io/leaning-grpc/) 查看html内容.