# JSON 映射

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [JSON Mapping](https://developers.google.com/protocol-buffers/docs/proto3#json) 一节

Proto3支持JSON格式的标准编码, 让在系统之间分享数据变得容易. 编码在下面的表格中以type-by-type的基本原则进行描述.

如果一个值在json编码的数据中丢失或者它的值是null, 在被解析成protocol buffer时它将设置为对应的默认值.如果一个字段的值正好是protocol buffer的默认值, 这个字段默认就不会出现在json编码的数据中以便节约空间.具体实现应该提供选项来在json编码输出中出现带有默认值的字段.

> 注: 详细表格不翻译, 具体见[原文](https://developers.google.com/protocol-buffers/docs/proto3#json).