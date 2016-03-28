Messages/消息
=========

> 注：内容翻译自官网文档 [Packages](https://developers.google.com/protocol-buffers/docs/reference/java-generated#message)

给出一个简单的消息定义：

	message Foo {}

protocol buffer 编译器生成名为 Foo 的类，实现 Message 接口。这个类被定义为 final， 不容许任何子类。Foo 继承自 GeneratedMessage， 但是这个可以认为是实现细节。默认， Foo 用为实现最大速度的特别版本来覆盖很多 GeneratedMessage 的方法。不过，如果 .proto 文件包含这行：

	option optimize_for = CODE_SIZE;

则 Foo 将只覆盖功能必要方法的最小集合，而其他方法依赖 GeneratedMessage 的基于反射的实现。这显著的降低了生成代码的大小，但是也降低了性能。要不然，如果 .proto 文件包含：

	option optimize_for = LITE_RUNTIME;

那么 Foo 将包含所有方法的快速实现，但是将实现 MessageLite 接口，只包含






