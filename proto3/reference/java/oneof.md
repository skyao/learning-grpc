Oneof
=========

> 注：内容翻译自官网参考文档中 Java Generated Code 的 [oneof](https://developers.google.com/protocol-buffers/docs/reference/java-generated#oneof) 一节。

给假设有一个类似这样的oneof定义：

```java
oneof oneof_name {
    int32 foo_int = 4;
    string foo_string = 9;
    ...
}
```

在 oneof_name 这个 oneof 中的所有字段将为他们的值使用共享的 oneof_name 对象。此外，protocol buffer compiler 将为每个oneof case生成一个Java枚举类型，如下所示：

```java
public enum OneofNameCase implements com.google.protobuf.Internal.EnumLite {
    FOO_INT(4),
    FOO_STRING(9),
    ...
    ONEOF_NAME_NOT_SET(0);
    ...
};
```

这个枚举类型的值有下列特殊方法：

- int getNumber(): 返回对象在.proto文件中定义的编号
- static Foo valueOf(int value): 返回对应给定编号的枚举对象，对于其他编号则返回null。

编译器也将在消息类和它的builder中生成下列访问方法：

- OneofNameCase getOneofNameCase(): 返回枚举指示哪个字段被设置。如果一个都没有设置返回 ONEOF_NAME_NOT_SET

编译器将仅在消息的builder中生成下列方法：

- Builder clearOneofName(): 设置 oneof_name 为null，并设置 oneof case 为ONEOF_NAME_NOT_SET
