类DemoServiceBlockingStub
========================

## 类定义

这个类是通过grpc的proto编译器生成的类，它的package由.proto文件中的 java_package 选项指定，如：

> option java_package = "io.grpc.examples.demo";

类定义如下，继承AbstractStub，并实现DemoServiceBlockingClient接口：

```java
package io.grpc.examples.demo;

@javax.annotation.Generated("by gRPC proto compiler")
public static class DemoServiceBlockingStub extends io.grpc.stub.AbstractStub<DemoServiceBlockingStub>
  implements DemoServiceBlockingClient {
}
```

## 构造函数

构造函数非常简单，直接调用基类 AbstractStub 的构造函数，传入Channel和CallOptions：

```java
    private DemoServiceBlockingStub(io.grpc.Channel channel) {
      super(channel);
    }

    private DemoServiceBlockingStub(io.grpc.Channel channel,
        io.grpc.CallOptions callOptions) {
      super(channel, callOptions);
    }
}
```

## 方法

首先实现了基类 AbstractStub 的抽象方法build()，只是简单的构造了自身的一个新的实例:

```java
@java.lang.Override
protected DemoServiceBlockingStub build(io.grpc.Channel channel,
    io.grpc.CallOptions callOptions) {
	return new DemoServiceBlockingStub(channel, callOptions);
}
```

### 业务方法实现

然后就是实现业务定义的service方法了，这里是定义在 DemoServiceBlockingClient 接口中。先看 DemoServiceBlockingClient 接口的定义：

```java
public static interface DemoServiceBlockingClient {
    public io.grpc.examples.demo.LoginResponse login(io.grpc.examples.demo.LoginRequest request);
    ......
}
```

这些方法对应于.proto文件中service定义的rpc方法：

```bash
service DemoService {
    rpc login (LoginRequest) returns (LoginResponse) {}
    ......
}
```

DemoServiceBlockingStub 中一一实现这些服务方法：


```java
@java.lang.Override
public io.grpc.examples.demo.LoginResponse login(io.grpc.examples.demo.LoginRequest request) {
      return blockingUnaryCall(getChannel(), METHOD_LOGIN, getCallOptions(), request);
}
```