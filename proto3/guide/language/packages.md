## 包

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Packages](https://developers.google.com/protocol-buffers/docs/proto3#packages) 一节

可以在.proto文件中增加一个可选的包标记来防止protocol消息类型之间的名字冲突.

```java
package foo.bar;
message Open { ... }
```

在定义消息类型的字段时可以这样使用包标志:

```java
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

包标志对生成代码的影响依赖所选的语言:

- 在c\++中, 生成类包裹在c\++ namespace中. 例如, Open将在namespace foo::bar中.
- 在java中, 包被作为java package使用, 除非你在.proto文件中显式提供java_package选项.
- 在Python中, 这个包指令将被忽略, 因为Python模块是根据他们在文件系统中的位置被组织的.
- 在Go中, 包被作为Go package使用, 除非你在.proto文件中显式提供go_package选项.
- 在Ruby中, 生成的代码被包裹在内嵌的Ruby namespaces, 转换为要求的Ruby capitalization风格(第一个字符大写;如果第一个字符不是字母则加一个PB_前缀). 例如, Open将在namespace Foo::Bar中.
- 在JavaNano中, 包被作为java package使用, 除非你在.proto文件中显式提供java_package选项.


## 包和命名解析

在protocol buffer语言中, 类型命名解析类似 C++: 首先搜索最内层的范围, 然后最内层的外一层, 以此类推, 每个包都被认为是它父包的"内层". 在最前面的'.'(例如, .foo.bar.Baz)意味着替换为从最外层范围开始.

protocol buffer编译器通过解析导入的.proto文件来解析所有的类型命名. 每个语言的代码生成器知道在这个语言中如何引用每个类型, 即使它有不同的范围规则.

