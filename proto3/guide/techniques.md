技巧
=======

> 注: 本文内容翻译来自Protocol Buffer 官网 Developer Guide 中的 [Techniques](http://groups.google.com/group/protobuf)一文.

这个页面讲述一些处理protocol buffer的被广泛应用的设计模式. 你也可以到[Protocol Buffers discussion group](http://groups.google.com/group/protobuf)发表设计和使用的问题.

# 流化多个消息

如果想写多个消息到单个文件或者流中, 你需要自己保持消息开始和结束的记录. Protocol buffer wire 格式不是自限定(self-delimiting), 所以protocol buffer解析器不能自己检测到消息的结束. 解决这个问题的最简单的方法是在写入消息自身之外写入每个消息的大小. 当读取消息回来的时候, 先读大小, 然后读取字节到分离的buffer, 再从buffer解析. (如果你想避免复制字节到分离的buffer, 检出 CodedInputStream 类(在c++和java中都有), 它可以被告之限制对特定数量字节的读取. 注: 这里有点不太理解, 不过不管了, zero-copy的特性而已)

# 大数据集

Protocol Buffer 并非设计用来处理巨型消息. 作为一个常用规则, 如果你处理每个都大于1M的消息, 是时候考虑使用交替策略了(alternate strategy).

这里说的是, Protocol Buffer 非常善于处理在一个大数据库集下的单个消息. 通常, 大数据集仅仅是小片段的集合, 这里每个小片段是数据的一个结构化的片段. 虽然Protocol Buffer不能一次性的处理整体, 但是使用Protocol Buffer来编码每个片段可以极大的简化问题: 现在需要做的是处理字节字符串的集合而不是结构体的集合.

Protocol Buffer 没有包括任何内建的对大数据集的支持, 因为不同的情况需要不同的解决方案. 某些时候一个简单的记录列表就可以搞定, 而其他时候可能想要更多东西比如数据库. 每个解决方案应该作为独立类库来开发, 这样仅仅是那些需要使用他们的人才需要付出代价.

# 联合类型

可能某些时候你想发送一个消息, 它可能是介个不同类型中的一个. 但是, protocol buffer解析器不能单独依靠消息的内容来检测消息的类型. 因此你如果确保接收的应用知道如何解码你的消息? 一个解决方式是创建一个包裹消息, 每个可能的消息类型有一个可选字段.

例如, 如果有你消息类型Foo, Bar, 和 Baz, 你可以这样用一个类型来结合他们:

```java
message OneMessage {
  // One of the following will be filled in.
  optional Foo foo = 1;
  optional Bar bar = 2;
  optional Baz baz = 3;
}
```

你可能也想有一个枚举字段可以用来标识哪个消息被填写了, 因此你可以据此做切换:

```java
message OneMessage {
  enum Type { FOO = 1; BAR = 2; BAZ = 3; }

  // Identifies which field is filled in.
  required Type type = 1;

  // One of the following will be filled in.
  optional Foo foo = 2;
  optional Bar bar = 3;
  optional Baz baz = 4;
}
```

如果你的可能类型数量非常巨大, 在包含类型里列出他们中的每一个会变得非常庞大, 你可以考虑使用[extensions](https://developers.google.com/protocol-buffers/docs/proto.html#extensions):

```java
message OneMessage {
  extensions 100 to max;
}
// Elsewhere...
extend OneMessage {
  optional Foo foo_ext = 100;
  optional Bar bar_ext = 101;
  optional Baz baz_ext = 102;
}
```

注意你能使用ListFields 反射方法(在C++, Java, 和 Python中) 来得到在这些消息中的所有字段的列表. 你可以用这个作为scheme的一部分为各式各样的消息类型注册处理器.

# 自描述消息

Protocol Buffers 没有包含他们自己类型的描述. 这样, 仅仅给出一个原始的消息而不带对应的定义它类型的.proto文件, 是很难提取出任何有用数据的.

然而, 注意.proto文件的内容可以通过使用protocol buffers来自我展现. 源文件包中的文件src/google/protobuf/descriptor.proto 定义了涉及的消息类型. protoc可以通过使用--descriptor_set_out选项输出一个FileDescriptorSet - FileDescriptorSet展现了多个.proto文件的集合. 你可以像这样定义一个自描述protocol 消息:

```java
message SelfDescribingMessage {
  // Set of .proto files which define the type.
  required FileDescriptorSet proto_files = 1;

  // Name of the message type.  Must be defined by one of the files in
  // proto_files.
  required string type_name = 2;

  // The message data.
  required bytes message_data = 3;
}
```

通过使用类似DynamicMessage的类(C++ 和 Java可用), 你可以编写工具来来操作SelfDescribingMessages.

前面所说的, 这个功能没有包含在protocol buffer类库中的原因是因为在google我们从来没有在google内部使用它.
