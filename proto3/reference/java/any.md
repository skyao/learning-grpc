Any
=========

> 注：内容翻译自官网参考文档中 Java Generated Code 的 [Any](https://developers.google.com/protocol-buffers/docs/reference/java-generated#any) 一节。

假设有一个类似这样的Any字段：

```java
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  google.protobuf.Any details = 2;
}
```

在我们生成的代码中，details字段的getter返回一个com.google.protobuf.Any的实例。这提供下面特殊方法来打包和拆包Any的值：

```java
class Any {
  // 打包给定的消息到一个Any中，使用默认类型URL前缀 “type.googleapis.com”.
  public static Any pack(Message message);
  // 打包给定消息到一个Any中，使用给定类型URl前缀
  public static Any pack(Message message,
                         String typeUrlPrefix);

  // 检查这个Any消息的负载(payload)是不是给定的类型
  public <T extends Message> boolean is(class<T> clazz);

  // 解包Any到给定的消息类型。如果类型不匹配或者解析负载失败则抛出异常
  public <T extends Message> T unpack(class<T> clazz)
      throws InvalidProtocolBufferException;
}
```
