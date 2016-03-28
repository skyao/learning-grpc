编译器调用
=========

> 注：内容翻译自官网文档 [Compiler Invocation](https://developers.google.com/protocol-buffers/docs/reference/java-generated#invocation)

当使用--java_out= 命令行标记时，protocol buffer编译器生成java输出。--java_out= 选项的参数是想编译器写java输出的目录。编译器为每个.proto文件输入创建一个单一的.java文件.这个文件包含一个单一的outer class定义，包含一些内嵌类和静态字段，基于.proto文件中的定义。

outer class的名字如下可选：如果.proto文件包含如下的一行：

	option java_outer_classname = "Foo";

则outer class的名字将会是 Foo。否则，outer class名字通过转换.proto文件的基本名称为驼峰法来决定。例如，foo_bar.proto将变成FooBar。如果在文件中有相同名字的消息，"OuterClass"将被附加到outer class的名字后。例如，如果 foo_bar.proto 包含一个名为 FooBar 的消息，outer class将变成 FooBarOuterClass 。

Java package名字的选择在后面的 [Packages](packages.md) 一节中描述。

通过连接参数到 --java_out= 来选择输出文件， package 名称(使用/替代.)和 .java 文件名。

因此，例如，如下调用编译器：

	protoc --proto_path=src --java_out=build/gen src/foo.proto

如果 foo.proto 的java package是 com.example 并且它的outer class名称是 FooProtos，那么protocol buffer编译器将生成文件 `build/gen/com/example/FooProtos.java`。protocol buffer编译器在需要时将自动创建 build/gen/com 和 build/gen/com/example 目录。但是，它将不会创建 build/gen 或者 build 。他们必须已经存在。可以在一次调用中指定多个 .proto文件；所有输出文件将被一次性生成。

==当输出java代码时，protocol buffer编译器直接输出到jar包的能力是非常方便的，因为很多java工具可以直接从jar文件中读取源代码。要输出到jar文件，简单的提供以.jar结束的输出位置。注意，仅有java源代码被放置在包中；还是必须单独编译来生成java class文件==



