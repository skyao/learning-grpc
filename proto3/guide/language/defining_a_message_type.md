# 定义消息类型

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Defining A Message Type](https://developers.google.com/protocol-buffers/docs/proto3#simple) 一节

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

## 定义字段类型

上面例子中, 所有字段都是Scalar Types/简单类型(下面会详细讲述), 包括两个int和一个string. 此外还可以支持复合类型包括枚举类型.

## 指定标签

可以看到消息定义的每个字段都有一个唯一的数字型标签. 这个标签用于在消息的二进制格式中标识字段, 一旦消息类型被使用后不可以再修改.

注意标签的值在1和15之间时编码只需一个字节, 包括标识值和字段类型(可以在Protocol Buffer Encoding中找到更多信息). 标签在16到2047之间将占用两个字节. 因此应该将从1到15的标签分派给最频繁出现的消息元素. 记得保留一些空间给未来可能添加的频繁出现的元素.

可以指定的最小的标签值是1, 最大的是2的29次方-1, 即536870911. 另外19000到19999(FieldDescriptor::kFirstReservedNumber through FieldDescriptor::kLastReservedNumber)不能使用, 因为Protocol Buffers的实现自己保留这个标签段.

## 指定字段规则

消息字段可以是下列之一:

1. 单数: 一个定义良好的消息可以有0个或1个此字段(但是不能超过1个).
2. 重复: 这个字段可以在定义良好的消息中重复任意次(包括0次).重复值的顺序将被维持原状.

由于历史原因, 简单数字类型的重复字段并没有编码为最有效率的方式. 新的代码应该使用特别选项[packed=true]来得到更有效率的编码. 例如:

	 repeated int32 samples = 4 [packed=true];

可以在Protocol Buffer Encoding中找到更多的关于packed encoding的信息.

## 添加更多消息类型

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

## 添加注释

可以使用C/C++风格的"//"语法在.proto文件中添加注释.

```java
message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

## 保留字段

当你更新消息类型,需要彻底删除一个字段时, 或者注释掉它, 未来的用户在实现他们对这个类型的更新时可以重用这个标签数字. 这将导致言中的问题, 如果他们后来装载同一个.proto文件的老版本, 败落数据冲突, 隐私缺陷以及其他. 为了确保不发生这样的事情, 有一个方法时指明这个要删除的字段为保留字段. 如果未来任何用户试图私用这个字段标识符protocol buffer的编译器就是告警.

```java
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

注意: **不能在同一个保留字段声明中混合使用名字和标签数字**

## .proto可以生成什么?

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

> 注： proto3版本的API参考手册已经放出，[点击这里查看中文翻译版本](../../reference/index.md)
