Protocl Buffer 3 语言指南
========================

> 注: 内容来自官网资料 [Language Guide (proto3)](https://developers.google.com/protocol-buffers/docs/proto3).

这份指南描述如何使用protocol buffer语言来构建你的protocol buffer数据,包括.proto文件语法和如何从.proto文件生成数据访问类. 覆盖protocol buffers语言的proto3版本, 对于老版本proto2的语法,请参考[Proto2语言指南](https://developers.google.com/protocol-buffers/docs/proto).

这是一个参考指南 - 如果需要逐步渐进的,使用到这份文档描述的众多特性的范例, 请见你所选语言的[教程](https://developers.google.com/protocol-buffers/docs/tutorials).

> 说明：这个文档的英文原文是放在一个页面中，实在有点长，阅读起来比较累，因此我翻译时按照章节拆分为多个页面。

拆分之后的章节列表：

- 定义消息类型
- Scalar值类型
- 默认值
- 枚举
- 使用其他消息类型
- 内嵌类型
- 更新消息类型
- Any
- Oneof
- Maps
- 包
- 定义服务
- JSON 映射
- 选项
- 生成类

