Protocol Buffer概述
========

> 注: 内容翻译来自官网文档 [Developer Guide Overview](https://developers.google.com/protocol-buffers/docs/overview).

欢迎来到protocol buffer的开发者文档 - protocol buffer是一个**语言无关，平台无关，可扩展的结构化数据序列化方案, 用于协议通讯, 数据存储和其他更多用途**.

这份文档针对想在应用中使用protocol buffer的Java, C++或者Python开发者。概述介绍protocol buffer并告诉你需要知道哪些来开始 - 你可以随后继续跟随 [教程](https://developers.google.com/protocol-buffers/docs/tutorials) 或者深入 [protocol buffer编码](encoding.md) 。所有三个语言的 [API参考文档](../reference/index.md) 也已经提供，还有编写.proto文件的 [语言](language_guide.html) 和 [风格](style_guide.html) 指南。

# protocol buffers是什么?

Protocol buffer 是一个灵活,高效,自动化的结构化数据序列化机制 - 想象xml, 但是更小, 更快并且更简单.一旦定义好数据如何构造, 就可以使用特殊的生成的源代码来轻易的读写你的结构化数据到和从不同的数据流，用不同的语言.你甚至可以更新你的数据结构而不打破已部署的使用"旧有"格式编译的程序.

# How do they work?

通过在.proto文件中定义protocol buffer消息类型来指定要序列化的信息如何组织。每个protocol buffer信息是一个小的信息逻辑记录，包含一序列的"名字-值"对。这里是一个非常基本的例子，.proto文件定义了一个消息，包含一个人的消息：

```java
message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}
```

如你所见，消息格式很简单 - 每个消息类型有一个或者多个唯一的编号的字段，而每个字段有一个名字和值类型，这里的值类型可以是数字（整型或者浮点），布尔，字符串，原始字节（raw bytes），或者甚至（如上面的例子）是其他protocol buffer消息，容许分层次的构建数据。可以指定可选字段，必填字段和重复字段。关于编写.proto文件的更多信息在 [protocol buffer 语言指南](language_guide.html) 中。

一旦定义了消息，可以在.proto文件上运行对应应用语言的protcol buffer的编译器来生成数据访问类。这些类为每个字段(类似name()或者set_name())提供简单的访问器，还有用于序列化/解析整个结构到/从原始字节的方法 - 因此，例如， 如果你选择的语言是c++，在上面的例子上运行编译器将会生成名为Person的类。然后可以用这个类在应用中获取，序列化，并获取 Person protocol buffer消息。可能随后编写一些类似这样的代码：

```java
Person person;
person.set_name("John Doe");
person.set_id(1234);
person.set_email("jdoe@example.com");
fstream output("myfile", ios::out | ios::binary);
person.SerializeToOstream(&output);
```

然后，稍后，可以这样读回消息：

```java
fstream input("myfile", ios::in | ios::binary);
Person person;
person.ParseFromIstream(&input);
cout << "Name: " << person.name() << endl;
cout << "E-mail: " << person.email() << endl;
```

可以添加新的字段到消息格式中，而不破坏向后兼容；老的二进制在解析时简单的忽略新的字段。因此，如果有一个使用protocol buffer作为数据格式的通讯协议，可以扩展协议而不必担心打破已有代码。

在 [API参考文档的章节](../reference/index.html) 中可以找到使用生成的protocol buffer代码的完整参考。

# 为什么不使用xml?

相比xml, Protocol buffer在序列化结构化数据方面有很多优势:

- 更简单
- 小3 到 10 倍
- 快 20 到 100 倍
- 更清晰
- 生成数据访问类, 更容易编程使用

例如，假设想要用 name 和 email 来构建一个 Person。在XML中，需要这样做：

```xml
<person>
    <name>John Doe</name>
    <email>jdoe@example.com</email>
</person>
```

而对应的protocol buffer消息(使用protocol buffer [文本格式](https://developers.google.com/protocol-buffers/docs/reference/cpp/google.protobuf.text_format)):

```javascript
# protocol buffer的文本展示
# 这 *不是* 实际使用的二进制格式。
person {
  name: "John Doe"
  email: "jdoe@example.com"
}
```

当这个消息被编码为protocol buffer 二进制格式(上面的文本格式仅仅是在调试和编辑时方便人阅读的表示方式)，它将可能是长28个字节并花费100-200纳秒来解析。XML版本至少需要69个字节，如果删除空白字符，并将话费5000 - 10000 纳秒来解析。

另外，操作protocol buffer也更简单：

```java
cout << "Name: " << person.name() << endl;
cout << "E-mail: " << person.email() << endl;
```

而使用XML，将不得不做类似的事情：

```java
cout << "Name: "
   << person.getElementsByTagName("name")->item(0)->innerText()
   << endl;
cout << "E-mail: "
   << person.getElementsByTagName("email")->item(0)->innerText()
   << endl;
```

当然，Protocol buffer 也不总是比XML更合适 - 例如，Protocol buffer 不适合建模基于文本的标志(如HTML)文档，因为无法轻易的使用文本交替结构。此外，XML是human-readable 和 human-editable的。Protocol buffer，至少他们原生的格式不是。XML也是某种程度上的自描述。Protocol buffer只有当有消息定义(.proto文件)时才有意义。

# 听起来不错！如何开始？

[下载包](https://developers.google.com/protocol-buffers/docs/downloads.html) - 包含Java,Python和C++protocol buffer 编译器的完整代码，还有需要用于I/O和测试的类。要构建和安装编译器，请遵循README中的建议。

# proto3介绍

我们 [最新的version 3 alpha release](https://github.com/google/protobuf/releases) (注：截至2016-04-01，最新版本是3.0.0-beta-2)带来了一个新的语言版本 - Protocol Buffers 语言版本 3 (又叫做 proto3)，在我们现有的语言版本(又叫做proto2)上增加了一些新特性. Proto3 简化了protocol buffer语言, 使其容易使用并可以用于更大的编程语言范围. 当前alpha版本可以生成Java, c++, Python, Javanano，Ruby，Objective-C, 和 C#(有一些限制)。此外还可以使用最新的Go protoc插件，在 [golang/protobuf](https://github.com/golang/protobuf) github仓库中 ,来生成proto3代码.

我们当前推荐仅仅在以下行和试用proto3:

1. 在proto3新支持的语言中使用protocol buffer
2. 使用新的开源RPC实现 [grpc](http://github.com/grpc/grpc-common) (当前也是alpha版本)(注：截至2016-4-1，grpc已经是beta版本) - 我们推荐所有新的gRPC服务器和客户端使用proto3，因为它避免了兼容问题。

注意这两个语言版本的API并非完全兼容。为了避免给现有用户造成麻烦，我们将继续在新的protocol buffer发行版中支持以前的语言版本。

在 [release notes](https://github.com/google/protobuf/releases) 中可以看到和当前默认版本的主要差别，并在 [Proto3 语言指南](language_guide.html) 中学习proto3的语法。proto3的完全版本将很快到来！

(如果proto2和proto3的名字看上去有点困惑，这是因为我们最初开源protocol buffer时，它实际上是google的第二个语言版本 - 也以proto2被人所知。这也是为什么我们的开源版本数字从v2.0.0开始。)

# 历史二三事

protocol buffer最早在google被开发用来处理index服务器请求/应答协议。在protocol buffer之前，有一个请求和应答的格式用于处理编组/解组，并支持多个协议版本。这导致了一些非常丑的代码，如：

```java
if (version == 3) {
	...
} else if (version > 4) {
    if (version == 5) {
     ...
    }
	...
}
```

明确被格式的协议也复杂化了新协议版本的推出，因为开发者不得不在他们按开关来开始使用新版本之前，确保在请求的发起者和处理请求的实际服务器之间的所有服务器理解新协议。

protocol buffer被设计用来解决很多这样的问题：

- 可以轻易引入新的字段，而不需要检查这些数据的中介服务器可以简单的解析并传递数据，而无需知道所有字段
- 格式更加自我描述，并可以处理多个语言 (C++, Java, 等)

然而，用户依然需要硬编码他们自己的解析代码。

随着系统的发展，要求一些其他特性和用法：

- 自动生成序列化和反序列化代码，避免硬解析的要求
- 除了用于短期RPC(Remote Procedure Call)请求外，人们开始使用protocol buffer来作为一个方便的自我描述的格式来存储持久化数据(例如在Bigtable中)。
- 服务器RPC接口开始定义为protocol文件的一部分，使用protocol 编译器生成用户可以用服务器的实际接口来覆盖的桩类。

Protocol buffer现在是google的数据通用语言 - 在写这个的时候，有 48162 个不同的消息类型被定义在google 代码树种，跨越 12183个 .proto文件。他们被用于RPC系统和在很多系统中的数据持久化存储。

> 注：原文最后更新于2016-1-20，翻译内容最后更新于2016-3-28.
