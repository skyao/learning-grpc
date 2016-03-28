Packages/包
=========

> 注：内容翻译自官网文档 [Packages](https://developers.google.com/protocol-buffers/docs/reference/java-generated#package)

生成的类被放置在一个基于 java_package 选项的java package中。如果这个选项缺失，将替代使用 package 定义。

例如，如果 .proto 文件包含：

	package foo.bar;

那么结果的java类将放置在Java package foo.bar中。不过，如果 .proto 文件也包含一个 java_package 选项，像这样：

	package foo.bar;
	option java_package = "com.example.foo.bar";

那么类将放置在 com.example.foo.bar package中。提供 java_package 选项是因为通常 .proto package 定义不会以倒置的域名来开始。

