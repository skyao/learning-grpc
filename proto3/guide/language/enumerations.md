枚举
======

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Enumerations](https://developers.google.com/protocol-buffers/docs/proto3#enum) 一节

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

枚举常量必须在32位整形的范围内. 由于枚举值使用varint encoding, 负值是效率低下的因此不推荐使用.你可以在消息定义中定义枚举, 如前面例子那样, 或者在外部 - 这些枚举可以在.proto文件的任意消息定义中重用. 你也可以用在一个消息中声明的枚举类型作为别的消息的字段类型, 需要使用语法MessageType.EnumType.

当你运行protocol buffer 编译器处理使用枚举的.proto文件时, 生成的代码将会有java或c++的对应枚举, 对于Python, 在生成的运行时类中会有一个特别的EnumDescriptor用于创建带有整型值的symbolic常量集合.

在反序列过程中, 未被识别的枚举值将被保留在消息中, 但是当消息被反序列号时将会如何表现是和语言有关的. 在支持开放枚举类型可以用定义范围之外的值的语言中, 例如C++和Go, 未知的枚举值被简单保存为它底层整型描述. 在封闭枚举类型的语言例如Java中, 一个枚举的特例用于表示这个未识别的值, 底层整型可以被特殊的访问器访问. 在其他案例中, 如果消息被系列化, 这个未识别的值将和消息一起被序列化.

注: 下面这段内容有点不能理解,翻译的不好, 以后理解清楚再来修订.

> In languages with closed enum types such as Java, a case in the enum is used to represent an unrecognized value, and the underlying integer can be accessed with special accessors. In either case, if the message is serialized the unrecognized value will still be serialized with the message.

关于如何在应用中处理消息枚举的更多信息, 请看所选语言的生成代码指南(generated code guide).