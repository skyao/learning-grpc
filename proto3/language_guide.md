Protocl Buffer 3 语言指南
========================

注: 内容翻译来自官网资料 [Language Guide (proto3)](https://developers.google.com/protocol-buffers/docs/proto3).

这份指南描述如何使用protocol buffer语言来构建你的protocol buffer数据,包括.proto文件语法和如何从.proto文件生成数据访问类. 覆盖protocol buffers语言的proto3版本, 对于老版本proto2的语法,请参考[Proto2语言指南](https://developers.google.com/protocol-buffers/docs/proto).

这是一个参考指南 - 如果需要逐步渐进的,使用到这份文档描述的众多特性的范例, 请见你所选语言的教程[教程](https://developers.google.com/protocol-buffers/docs/tutorials).

# 定义消息类型

首先我们来看一个非常简单的例子. 让我们假设你想定义一个搜索请求消息格式, 每个搜索请求有一个查询字符串, 搜索结果的第几页/每页结果数量. 这里是你用来定义消息类型的.proto文件:

```java
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

1. 第一行指明当前使用的是proto3语法, 如果不指定则默认为proto2. **必须是.proto文件的除空行和注释内容之外的第一行**
2. SearchRequest 消息定义指明了3个字段. 每个字段有名字和类型.

# 定义字段类型

上面例子中, 所有字段都是Scalar Types/简单类型(下面会详细讲述), 包括两个int和一个string. 此外还可以支持复合类型包括枚举类型.

# 指定标签

可以看到消息定义的每个字段都有一个唯一的数字型标签. 这个标签用于在消息的二进制格式中标识字段, 一旦消息类型被使用后不可以再修改.

注意标签的值在1和15之间时编码只需一个字节, 包括标识值和字段类型(可以在Protocol Buffer Encoding中找到更多信息). 标签在16到2047之间将占用两个字节. 因此应该将从1到15的标签分派给最频繁出现的消息元素. 记得保留一些空间给未来可能添加的频繁出现的元素.

可以指定的最小的标签值是1, 最大的是2的29次方-1, 即536870911. 另外19000到19999(FieldDescriptor::kFirstReservedNumber through FieldDescriptor::kLastReservedNumber)不能使用, 因为Protocol Buffers的实现自己保留这个标签段.

# 指定字段规则

消息字段可以是下列之一:

1. 单数: 一个定义良好的消息可以有0个或1个此字段(但是不能超过1个).
2. 重复: 这个字段可以在定义良好的消息中重复任意次(包括0次).重复值的顺序将被维持原状.

由于历史原因, 简单数字类型的重复字段并没有编码为最有效率的方式. 新的代码应该使用特别选项[packed=true]来得到更有效率的编码. 例如:

	 repeated int32 samples = 4 [packed=true];

可以在Protocol Buffer Encoding中找到更多的关于packed encoding的信息.

# 添加更多消息类型

可以在单个 .proto 文件中定义多个消息类型. 适用于定义多个相关消息, 例如, 如果你想定义应答消息格式来相应你的SearchResponse消息类型,你应该将它添加到同一个.proto文件.

```java
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

# 添加注释

可以使用C/C++风格的"//"语法在.proto文件中添加注释.

```java
message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

# 保留字段

当你更新消息类型,需要彻底删除一个字段时, 或者注释掉它, 未来的用户在实现他们对这个类型的更新时可以重用这个标签数字. 这将导致言中的问题, 如果他们后来装载同一个.proto文件的老版本, 败落数据冲突, 隐私缺陷以及其他. 为了确保不发生这样的事情, 有一个方法时指明这个要删除的字段为保留字段. 如果未来任何用户试图私用这个字段标识符protocol buffer的编译器就是告警.

```java
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

Note that you can't mix field names and tag numbers in the same reserved statement.

注意: **不能在同一个保留字段声明中混合使用名字和标签数字**

# .proto可以生成什么?

对.proto文件执行protocol buffer的编译器时, 编辑器会生成你所选语言的代码. 你可以用来处理你在文件中定义的消息类型, 包括读写字段值, 系列化消息到输出流, 和从输入流中解析消息.

- 对于c++, 编译器为每个.proto文件生成.h和.cc文件, 并为每个在文件中描述的消息类型生成一个类
- 对于Java, 编译器为每个消息类型生成一个.java文件, 还有用于创建消息实例的Builder类.
- Python小有不同 - Python编译器为.proto文件中的每个消息类型生成带有静态描述符的模块, 这些模块将和metaclass一起在运行时用来创建必须的Python数据访问类.
- 编译器生成一个.pb.go文件, .proto文件中的每个消息类型在这个文件中都有一个类型.
- 对于Ruby, 编译器生成一个.rb文件, 带用包含消息类型的Ruby模块.
- 对于JavaNano, 编译器输出类似java但是没有builder类.
- 对于Objective-C, 编译器从每个.proto文件生成一个pbobjc.h和pbobjc.m文件,文件中描述的每个消息类型都有一个类.
- 对于C#, 编译器从每个.proto文件生成一个.cs文件,文件中描述的每个消息类型都有一个类.

你可以通过所选语言的教程(proto3版本即将到来)来找到更多关于如何使用每个语言的API的信息. 对于API的细节, 请看相应的[API参考手册](https://developers.google.com/protocol-buffers/docs/reference/overview)(proto3版本即将到来).

# 简单(Scalar)值类型

注: scalar是标量的意思,但是在如果翻译为标量有点不知所云, 先翻译为简单类型试试.

A scalar message field can have one of the following types – the table shows the type specified in the .proto file, and the corresponding type in the automatically generated class:

简单消息字段可以有下列类型其中之一 - 下面的表格显示在.proto文件中指明的类型和在自动生成类中的相应类型.

注: 原表格太复杂, 翻译没有必要, 请见[原页面](https://developers.google.com/protocol-buffers/docs/proto3#scalar). 

下面是简化版本, 只列出了我关心的java类型映射:

| .proto Type   | Java Type     |
| ------------- |:-------------:|
| double        | double 		|
| float         | float      	|
| int32			| int 	        |
| int32			| int 	        |
| int64			| long 	        |
| bool			| boolean       |
| string		| String        |
| bytes			| ByteString    |

TBD: ByteString 是个什么东东?

你可以在[Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding)中找到更多资料, 了解当你序列化消息的时候, 这些类型是如何编码的.

# 默认值

When a message is parsed, if the encoded message does not contain a particular singular element, the corresponding field in the parsed object is set to the default value for that field. These defaults are type-specific:

当消息被解析时, 如果被编码的消息没有包含特定的简单元素, 被解析的对象对应的字段被设置为默认值. 默认值是和类型有关的:

- 对于strings, 默认值是空字符串(注, 是"", 而不是null)
- 对于bytes, 默认值是空字节(注, 应该是byte[0], 注意这里也不是null)
- 对于boolean, 默认值是false.
- 对于数字类型, 默认值是0.
- 对于枚举, 默认值是第一个定义的枚举值, 而这个值必须是0.
- 对于消息字段, 默认值是null.

对于重复字段, 默认值是空(通常都是空列表)

注意: 对于简单字段, 当消息被解析后, 是没有办法知道这个字段到底是有设置值然后恰巧和默认值相同(例如一个boolean设置为false)还是这个字段没有没有设置值而取了默认值. 例如, 不要用一个boolean值然后当设置为false时来切换某些行为, 而你又不希望这个行为默认会发生. 同样请注意: 如果一个简单消息字段被设置为它的默认值, 这个值不会被序列化.

# 枚举

当你定义消息类型时, 你可能希望某个字段只能有预先定义的多个值中的一个. 例如, 假设你想为每个SearchRequest添加一个corpus字段, 而corpus可以是UNIVERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS 或 VIDEO. 你可以简单的添加一个枚举到消息定义, 为每个可能的值定义常量.

在下面的例子中, 我们添加一个名为Corpus的枚举类型, 定义好所有可能的值, 然后添加一个类型为Corpus的字段:

```java
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

如你所见, Corpus 枚举的第一个常量设置到0: 每个枚举定义必须包含一个映射到0的常量作为它的第一个元素. 这是因为:

- 必须有一个0值, 这样我们才能用0来作为数值默认值.
- 0值必须是第一个元素, 兼容proto2语法,在proto2中默认值总是第一个枚举值

可以通过将相同值赋值给不同的枚举常量来定义别名. 为此需要设置allow_alias选项为true, 否则当发现别名时protocol编译器会生成错误消息.

```java
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```


