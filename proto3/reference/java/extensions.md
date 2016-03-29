扩展 (仅用于proto2)
=========

> 注：内容翻译自官网参考文档中 Java Generated Code 的 [Extensions](https://developers.google.com/protocol-buffers/docs/reference/java-generated#extension) 一节。

假设有一个消息带有扩展范围：

```java
message Foo {
	extensions 100 to 199;
}
```

protocol buffer编译器将让 Foo 继承 GeneratedMessage.ExtendableMessage 而不是通常的 GeneratedMessage. 类型的， Foo的builder将继承 GeneratedMessage.ExtendableBuilder 。你应该从不通过名字来引用这些基本类型(GeneratedMessage 被认为是实现细节)。然而，这些超类定义了一些额外的方法让你可以用来操作扩展。

尤其 Foo 和 Foo.Builder 将继承方法 hasExtension(), getExtension() 和 getExtensionCount()。此外， Foo.Builder 将继承方法 setExtension() 和 clearExtension()。每个这些方法都将获取一个扩展标识符(下面描述)，作为它们的第一个参数，用来标记一个扩展字段。剩下的参数和返回值和为同样类型的普通(非扩展)字段而生成的那些对应的访问器方法完全相同。

给出一个扩展定义：

```java
extend Foo {
	optional int32 bar = 123;
}
```

protocol buffer 编译器生成一个名为 bar 的"扩展标识符"，可以和 Foo的扩展访问器一起使用来访问这个扩展，像这样：

```java
Foo foo =
    Foo.newBuilder()
     .setExtension(bar, 1)
     .build();
assert foo.hasExtension(bar);
assert foo.getExtension(bar) == 1;
```

(扩展标识符的确切实现非常复杂而且涉及到范型的不可意思的(magical)的使用 - 不过，在使用时你不用担心扩展标识符是如何工作。)

注意 bar 将被声明为.proto文件的outer class的静态字段， 如 [前面描述](compiler_invocation.md) 的。在这个例子中，我们忽略了outer class 的名字。

扩展可以内嵌在其他类型中申明。例如，通用模式是这样：

```java
message Baz {
    extend Foo {
    	optional Baz foo_ext = 124;
    }
}
```

在这个案例中，扩展标识符 foo_ext 内嵌在 Baz 中声明。它可以像下面这样使用：

```java
Baz baz = createMyBaz();
Foo foo =
    Foo.newBuilder()
    .setExtension(Baz.fooExt, baz)
    .build();
```

当解析一个可能有扩展的消息时，必须提供一个 ExtensionRegistry ，在这里已经注册了需要能够解析的任何扩展。否则，那些扩展将被当成未知字段。例如：

```java
ExtensionRegistry registry = ExtensionRegistry.newInstance();
registry.add(Baz.fooExt);
Foo foo = Foo.parseFrom(input, registry);
```

