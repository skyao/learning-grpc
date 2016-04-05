# 定义服务

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Defining Services](https://developers.google.com/protocol-buffers/docs/proto3#services) 一节

如果想在RPC (Remote Procedure Call) 系统中使用消息类型, 可以在.proto文件中定义RPC服务接口, 然后protocol buffer编译器会生成所选语言的服务接口代码和桩(stubs). 因此, 例如, 如果想定义一个RPC服务,带一个方法处理SearchRequest并返回SearchResponse, 可以在.proto文件中如下定义:

```java
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

使用protocol buffers最直接的RPC系统是gRPC: 一个Google开发的语言和平台无关的开源RPC系统. gRPC 可以非常好和protocol buffers一起工作并使用特别的protocol buffer编译器插件从.proto文件直接生成对应的RPC代码.

如果不想用gRPC, 也可以在自己的RPC实现中使用protocol buffers. 可以在[Proto2语言指南](https://developers.google.com/protocol-buffers/docs/proto#services)中找到更多这个消息.

也有一个第三方项目正在为Protocol Buffers开发RPC实现. 这里有一份我们知道的项目列表, 请见[third-party add-ons wiki page](https://github.com/google/protobuf/wiki/Third-Party-Add-ons).
