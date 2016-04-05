Scalar值类型
================

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Scalar Value Types](https://developers.google.com/protocol-buffers/docs/proto3#scalar) 一节

> 注: scalar是标量的意思,但是在如果翻译为标量有点不知所云, 先翻译为简单类型试试.

简单消息字段可以有下列类型其中之一 - 下面的表格显示在.proto文件中指明的类型和在自动生成类中的相应类型.

> 注: 原表格太复杂, 翻译没有必要, 请见[原页面](https://developers.google.com/protocol-buffers/docs/proto3#scalar).

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

你可以在 [Protocol Buffer Encoding](../encoding.md) 中找到更多资料, 了解当你序列化消息的时候, 这些类型是如何编码的.
