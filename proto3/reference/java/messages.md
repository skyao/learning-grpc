消息
=========

> 注：内容翻译自官网文档 [Messages](https://developers.google.com/protocol-buffers/docs/reference/java-generated#message)

给出一个简单的消息定义：

	message Foo {}

protocol buffer 编译器生成名为 Foo 的类，实现 Message 接口。这个类被定义为 final， 不容许任何子类。Foo 继承自 GeneratedMessage， 但是这个可以认为是实现细节。默认， Foo 用为实现最大速度的特别版本来覆盖很多 GeneratedMessage 的方法。不过，如果 .proto 文件包含这行：

	option optimize_for = CODE_SIZE;

则 Foo 将只覆盖功能必要方法的最小集合，而其他方法依赖 GeneratedMessage 的基于反射的实现。这显著的降低了生成代码的大小，但是也降低了性能。再不然，如果 .proto 文件包含：

	option optimize_for = LITE_RUNTIME;

那么 Foo 将包含所有方法的快速实现，但是将实现 MessageLite 接口，MessageLite 接口只包含 Message 的方法子集。尤其，它不支持描述符和反射。但是，在这种模式下，生成的代码仅需要链接到libprotobuf-lite.jar而不是libprotobuf.jar。这个"lite"类库比完全类库小很多，而且更加适合资源受限系统例如移动手机。

Message 接口定义方法让你检查，操纵，读取或者写入完整的消息。在这些方法之外， Foo class定义了下列静态方法：

- static Foo getDefaultInstance(): 返回Foo的单例实例，这等效于调用Foo.newBuilder().build() (这样所有字段被重置而所有重复字段清空)。注意消息的默认实例可以通过调用它的newBuilderForType()用来作为工厂(factory)。
- static Descriptor getDescriptor(): 返回类型的描述符。这包含关于类型的信息，包括它有什么字段和他们的类型是什么。这可以和消息的反射方法一起使用，例如 getField().
- static Foo parseFrom(...): 从给定的资源解析类型 Foo 的消息并返回它。在Message.Builder接口中的mergeFrom()的每个变量有一个对应的parseFrom方法。注意parseFrom()从来不抛出UninitializedMessageException; 如果被解析的消息缺失必要的字段它会抛出InvalidProtocolBufferException。这使得它和调用Foo.newBuilder().mergeFrom(...).build()有微妙的不同。
- static Parser parser(): 返回解析器实例，它实现了多个parseFrom()方法
- Foo.Builder newBuilder(): 创建一个新的builder(下面描述).
- Foo.Builder newBuilder(Foo prototype): 创建一个新的builder，所有字段初始化为和在prototype中一样的值。因为内嵌消息和字符串对象是不可变，他们将会在原始对象和复制对象之间共享。

## Builders

Message 对象 - 例如上面描述的 Foo class的实例 - 是不可变的，就像Java字符串。为了构建一个消息对象，需要使用builder。每个message类有它自己的bilder类 - 因此在我们的Foo例子中，protocol buffer编译器生成了一个内嵌类 Foo.Builder 可以用来构建Foo。Foo.Builder实现Message.Builder接口。它继承自GeneratedMessage.Builder类，但是，再次，这应该视为实现细节。和Foo类似，Foo.Builder 可能依靠范型方法实现，或者，当optimize_for选项被使用时，生成的定制化代码要快的多。

Foo.Builder没有定义任何静态方法。它的接口和通过Message.Builder定义的完全一样，不同的是返回类型更加明确：Foo.Builder的方法修改了builder返回类型Foo.Builder，而build()返回类型Foo。

builder的修改内容的方法 - 包括字段setter - 通常返回builder的引用(例如，他们"return this;")。这容许多个方法调用可以在一行中链起来。例如：

	builder.mergeFrom(obj).setFoo(1).setBar("abc").clearBaz();

## Sub Builders

对于包含子消息的消息，编译器也生成子builder。这容许你反复修改深层嵌套的子消息而不用重新构建他们。例如：

```java
message Foo {
  optional int32 val = 1;
  // some other fields.
}

message Bar {
  optional Foo foo = 1;
  // some other fields.
}

message Baz {
  optional Bar bar = 1;
  // some other fields.
}
```

如果你已经有了一个Baz消息，并且想修改深度内嵌的在Foo中的val。以其：

If you have a Baz message already, and want to change the deeply nested val in Foo. Instead of:

```java
baz = baz.toBuilder().setBar(
    baz.getBar().toBuilder().setFoo(
      baz.getBar().getFoo().toBuilder().setVal(10).build()
    ).build()).build();
```

你可以写：

```java
Baz.Builder builder = baz.toBuilder();
    builder.getBarBuilder().getFooBuilder().setVal(10);
    baz = builder.build();
```

## 内嵌类型

消息可以在其他消息内部定义。例如: message Foo { message Bar { } }

在这种情况下，编译器简单的生成Bar作为内部类内嵌在Foo中。
