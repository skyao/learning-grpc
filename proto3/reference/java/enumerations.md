枚举
=========

> 注：内容翻译自官网参考文档中 Java Generated Code 的 [enumerations](https://developers.google.com/protocol-buffers/docs/reference/java-generated#enum) 一节。

假设有这样的枚举定义：

```java
enum Foo {
  VALUE_A = 0;
  VALUE_B = 5;
  VALUE_C = 1234;
}
```

protocol buffer 编译器将生成一个名为 Foo 的Java枚举类型，带有同样的值集合。如果正在使用proto3,还将添加特殊值 UNRECOGNIZED 到枚举类型。生成的枚举类型的值有下列特殊方法：

- int getNumber()：返回对象定义在.proto文件中的编号
- EnumValueDescriptor getValueDescriptor(): 返回枚举值的描述符，包含枚举值的信息如值的名字，编号和类型。
- EnumDescriptor getDescriptorForType(): 返回枚举类型的描述符，包含关于每个定义值的信息。

此外，Foo 枚举类型包含下列静态方法：

- static Foo forNumber(int value): 返回对应给定编号值的枚举对象。如果没有对应的枚举对象则返回null。
- static Foo valueOf(int value): 返回对应给定编号的枚举对象。这个方法被弃用，以forNumber(int value)方法替代，并将在即将出现的发布版本中移除。
- static Foo valueOf(EnumValueDescriptor descriptor): 返回对应给定枚举值描述符的枚举对象。可能比valueOf(int)快。在proto3中如果传递一个未知的枚举值描述符则会返回 UNRECOGNIZED
- EnumDescriptor getDescriptor(): 返回枚举类型的描述符，包含关于每个定义的枚举值的信息（和getDescriptorForType() 唯一的差别在于它是一个静态方法）

为每个枚举值生成一个带有 _VALUE 后缀的整型常量。

注意 .proto 语言容许多个枚举符号拥有同一个编号。有相同编号的符号是同义词。例如：

```java
enum Foo {
    BAR = 0;
    BAZ = 0;
}
```

在这个案例中， BAZ 是 BAR 的同义词。在Java中， BAZ 将被定义为一个像这样的static fianl 字段：

	static final Foo BAZ = BAR;

因此， BAR 和 BAZ 是相同的， 而 BAZ 应该从不出现在 switch 语句中。编译器通常选择用给定编号定义的第一个符号作为这个符号的"权威"版本。所有随后有同样编号的符号仅仅是别名。

枚举可以在消息类型中内嵌定义。编译器生成内嵌在消息类型类中的java枚举定义。
