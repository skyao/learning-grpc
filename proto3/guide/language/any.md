# Any

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Any](https://developers.google.com/protocol-buffers/docs/proto3#any) 一节

Any 消息类型可以让你使用消息作为嵌入类型而不必持有他们的.proto定义. Any把任意序列化后的消息作为bytes包含, 带有一个URL, 工作起来类似一个全局唯一的标识符. 为了使用Any类型, 需要导入google/protobuf/any.proto.

```java
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated Any details = 2;
}
```

给定消息类型的 default type URL 是 type.googleapis.com/packagename.messagename.

不同语言实现将提供运行时类库来帮助以类型安全的方式封包和解包Any的内容 - 例如, 在java中, Any类型将有特别的pack()和unpark()访问器, 而在c++中有PackFrom() 和 PackTo()方法:

```java
// 在Any中存储任务消息类型
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// 从Any中读取任意消息
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.IsType<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

**目前用于处理Any类型的运行时类库正在开发中**.

如果你熟悉proto2语法, Any 类型替代了extensions.