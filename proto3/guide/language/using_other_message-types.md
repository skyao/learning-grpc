使用其他消息类型
=============

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Using Other Message Types](https://developers.google.com/protocol-buffers/docs/proto3#other) 一节

可以使用其他消息类型作为字段类型. 例如, 假设你想在每个SearchResponse消息中包含Result消息 - 为了做到这点, 你可以在相同的.proto文件中定义Result消息类型然后具体指定SearchResponse中的一个字段为Result类型:

```java
message SearchResponse {
  repeated Result result = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

## 导入定义

在上面的例子中, Result消息类型是定义在和SearchResponse同一个文件中 - 如果你想要作为字段类型使用的消息类型已经在其他的.proto文件中定义了呢?

你可以通过导入来使用来自其他.proto文件的定义. 为了导入其他.proto的定义, 需要在文件的顶端增加导入声明:

> import "myproject/other_protos.proto";

默认只能从直接导入的.proto文件中使用其中的定义. 但是, 某些情况下可能需要移动.proto文件到新的位置. 相比直接移动.proto文件并更新所有的调用点, 现在可以有其他的方法: 在原有位置放置一个伪装的.proto文件, 通过使用import public方式转发所有的import到新的位置. 其他任何导入这个包含import public语句的proto文件都可以透明的得到通过import public方法导入的依赖. 例如:

```java
// new.proto
// 所有的依赖被移动到这里
```
```java
// old.proto
// 这里是所有client需要导入的proto
import public "new.proto";
import "other.proto";
```
```java
// client.proto
import "old.proto";
// 这里可以使用old.proto 和 new.proto 中的定义, 但是other.proto中的不能使用.
```

protocol编译器在通过命令行-I/--proto_path参数指定的目录集合中搜索导入的文件. 如果没有指定, 则在编译器被调用的目录下查找. 通常应该设置--proto_path参数到项目所在的根目录然后为所有的导入使用完整限定名.

## 使用proto2的消息类型

导入proto2的消息类型并在proto3消息中使用是可以的, 反之也如此. 但是, **proto2的枚举不能在proto3语法中使用**