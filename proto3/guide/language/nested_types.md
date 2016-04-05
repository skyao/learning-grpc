# 内嵌类型

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Nested Types](https://developers.google.com/protocol-buffers/docs/proto3#nested) 一节

可以在消息类型内部定义和使用消息类型, 如下面的例子所示 - 这里Result消息被定义在SearchResponse消息内部:

```java
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result result = 1;
}
```

如果想在父消息类型之外重用消息类型, 可以使用 Parent.Type 来引用:

```java
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

只要愿意可以内嵌的更深:

```java
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

