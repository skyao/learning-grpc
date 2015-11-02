gRPC概述
========

注: 官网文档[Guide Overview](http://www.grpc.io/docs/)的读书笔记, 这个文档水比较多, 不直接逐段翻译, 只摘录部分内容作为读书笔记.

一开始就如下介绍gRPC:

> a language-neutral, platform-neutral, open source, remote procedure call (RPC) system initially developed at Google.
>
> 一个语言无关, 平台无关, 开源的RPC系统, 最初由google开发

注: 和protocol buffer的介绍一样, 非常强调"语言无关, 平台无关".

What is gRPC?

# 什么是gRPC?

在gRPC中, 客户端应用可以直接调用在另外一个机器上运行的服务器应用, 就好像调用本地对象一样, 使得创建分布式应用和服务变得简单. 和很多RPC系统一样, gRPC是基于这样的思路: 定义服务, 指定可以被远程调用的方法, 带有参数和返回类型. 在服务器端, 服务器实现接口并运行一个gRPC服务器来处理客户端调用. 在客户端这边, 客户端有一个桩(stub)来提供和服务器端完全一样的方法.

![](http://www.grpc.io/img/grpc_concept_diagram_00.png)

# Working with protocol buffers

gRPC默认使用Protocol buffer, 版本为proto3. 但是据说gRPC也可以使用其他的序列化方案, 比如json. 注: 然后并没有看到到底如何用和哪个项目在用.

# Hello gRPC!

用一个最简单的例子来示范如何使用gRPC, 需要三个步骤.

- Create a protocol buffers schema that defines a simple RPC service with a single Hello World method.
- Create a server that implements this interface in your favourite language (where available).
- Create a client in your favourite language (or any other one you like!) that accesses your server.

## Install gRPC

好消息是, 对于java,不需要安装什么, 有jdk和grpc的依赖包就好了.

## 定义服务

在helloworld.proto文件中定义服务:

```java
syntax = "proto3";

option java_package = "io.grpc.examples";

package helloworld;

// The greeter service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

## 生成gRPC代码

具体如何生成代码后面再看, 先看生成的结果里面都有什么.

在这个example中生成的代码在src/generated/main中, 有两个目录:

1. grpc: 里面只有一个io/grpc/examples/helloworld/GreeterGrpc.java文件, 包含和rpc调用相关的代码, 比如service定义, 客户端stub的实现等.
2. java: 在目录io/grpc/examples/helloworld里面有多个java文件, HelloRequest.java和HelloResponse.java是从.proto文件中的message定义生成的message类. HelloRequestOrBuilder和HelloResponseOrBuilder是对应HelloRequest和java和HelloResponse的builder类. 然后还有一个HelloWorldProto.java, 里面保存了一些和.proto相关的信息.

## 生成的grpc代码

在GreeterGrpc.java的文件中, 在这个java文件中package定义为

	package io.grpc.examples.helloworld;

对应上面的.proto文件中的java_package和package两个设置, 似乎最终得到的java package为java_package + package?

    option java_package = "io.grpc.examples";
    package helloworld;

在GreeterGrpc.java中内嵌有多个java类.

### Greeter接口定义

- interface Greeter

    对应.proto文件中的service定义, 特别留意这里的sayHello()方法的定义,HelloResponse并不是简单的函数返回的方式.

	```java
	public static interface Greeter {
        public void sayHello(HelloRequest request, StreamObserver<HelloResponse> responseObserver);
    }
    ```

- GreeterBlockingClient和GreeterFutureClient

	提供和Greeter接口功能相同但是调用方式不一样的两种客户端调用接口.

	```java
    public static interface GreeterBlockingClient {
		// 这个是阻塞模式, 因此sayHello方法是用方法return的方式返回HelloResponse
    	public HelloResponse sayHello(HelloRequest request);
    }

    public static interface GreeterFutureClient {
		// future方式, 返回一个ListenableFuture
    	public ListenableFuture<HelloResponse> sayHello(HelloRequest request);
    }
    ```

### Greeter的客户端桩实现

- GreeterStub

	client端的stub实现类, 实现Greeter接口.

	```java
    public static class GreeterStub extends io.grpc.stub.AbstractStub<GreeterStub> implements Greeter { }
    ```

- GreeterBlockingStub 和 GreeterFutureStub

	client端的blocking stub实现类, 实现GreeterBlockingClient接口.

	```java
    public static class GreeterBlockingStub extends AbstractStub<GreeterBlockingStub> implements GreeterBlockingClient {}
    ```

	client端的future stub实现类, 实现GreeterFutureClient接口.

	```java
    public static class GreeterFutureStub extends AbstractStub<GreeterFutureStub> implements GreeterFutureClient {}
    ```

### 创建stub的方法

提供三个方法用来创建三种调用方法所需要的client stub.

```java
public static GreeterStub newStub(Channel channel) {
    return new GreeterStub(channel);
}

public static GreeterBlockingStub newBlockingStub(Channel channel) {
    return new GreeterBlockingStub(channel);
}

public static GreeterFutureStub newFutureStub(Channel channel) {
    return new GreeterFutureStub(channel);
}
```

### 其他代码内容

#### SERVICE_NAME常量定义

```java
public class GreeterGrpc {
	public static final String SERVICE_NAME = "helloworld.Greeter";
}
```

注意这个常量定义是public, 需要用到的时候应该可以直接调用或者反射调用来获取当前服务名.

#### 静态方法定义

从注释上看, "Static method descriptors that strictly reflect the proto", 这里是生成了静态方法描述以便以后做反射的时候使用.

```java
  // Static method descriptors that strictly reflect the proto.
  @io.grpc.ExperimentalApi
  public static final io.grpc.MethodDescriptor<io.grpc.examples.helloworld.HelloRequest,
      io.grpc.examples.helloworld.HelloResponse> METHOD_SAY_HELLO =
      io.grpc.MethodDescriptor.create(
          io.grpc.MethodDescriptor.MethodType.UNARY,
          generateFullMethodName(
              "helloworld.Greeter", "SayHello"),
          io.grpc.protobuf.ProtoUtils.marshaller(io.grpc.examples.helloworld.HelloRequest.getDefaultInstance()),
          io.grpc.protobuf.ProtoUtils.marshaller(io.grpc.examples.helloworld.HelloResponse.getDefaultInstance()));
```

#### bindService()方法

TBD: 还没有看懂这个方法的用途, 和在哪里调用, 后续看懂了再更新.

## 生成的java代码

在目录io/grpc/examples/helloworld里面的这些java代码:

- HelloRequest.java和HelloResponse.java是从.proto文件中的message定义生成的message类.
- HelloRequestOrBuilder和HelloResponseOrBuilder是对应HelloRequest和java和HelloResponse的builder类.
- HelloWorldProto.java, 里面保存了一些和.proto相关的信息.

message类和对应的Builder类的关系后面再具体看.
