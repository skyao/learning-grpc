Protocol Buffer概述
========

```uml
@startuml
a -> b
@enduml
```

注: 官网文档[Developer Guide Overview](https://developers.google.com/protocol-buffers/docs/overview)的读书笔记, 这个文档水比较多, 不直接逐段翻译, 只摘录部分内容作为读书笔记.

一开始就给出protocol buffers 的定义:

> a language-neutral, platform-neutral, extensible way of serializing structured data for use in communications protocols, data storage, and more.

> 语言无关, 平台无关, 可扩展的结构化数据序列化方案, 用于协议通讯, 数据存储和其他更多用途.

# protocol buffers是什么?

详细解释什么是protocol buffers:

1. Protocol buffer 是一个灵活,高效,自动化的结构化数据序列化机制 - 想到xml, 但是更小, 更快并且更简单.

2. 一旦你定义好你的数据如何构造, 然后你可以用不同的语言使用特殊的生成源代码来轻易的读写你的结构化数据到和从不同的数据流.

3. 你甚至可以更新你的数据结构而不打破已部署的使用"旧有"格式编译的程序.

# How do they work?

protocol buffer的工作方式简单说有以下几个步骤:

1. 通过在.proto文件中定义protocol buffer消息类型定义需要序列化的信息的结构
2. 按照应用语言的语言选择运行对应的protocol buffer 编译器处理.proto文件来生成数据访问类
3. 在应用中使用这些类来序列化和取回protocol buffer消息

# 为什么不使用xml?

相比xml, Protocol buffer在序列化结构化数据方面有很多优势:

- 更简单
- 小3 到 10 倍
- 快 20 到 100 倍
- 更清晰
- 生成数据访问类, 更容易编程使用

也提到Protocol buffer不适用的场合:

- 建模基于文本的标志语言(如HTML), 主要是不好结构化
- 不是human-readable 和 human-editable, Protocol buffer原生是二进制格式
- 不能自我描述, Protocol buffer必须依靠消息定义(如.proto文件)

# proto3介绍

Protocol buffer最新的版本是version 3 alpha. Proto3 简化了protocol buffer语言, 使其容易使用并可以用于更大的编程语言范围. 当前alpha版本支持Java, c++, Python, Javanano 和Ruby(有一些限制), 还有Go.

proto3目前只推荐在下面两种情况下使用:

1. 在proto3新支持的语言中使用protocol buffer
2. 使用新的开源RPC实现grpc(当前也是alpha版本)









