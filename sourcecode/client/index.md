gRPC客户端源码分析
===========

# 客户端调用流程

## 流程概述

标准的grpc client调用代码，最简单的方式，就三行代码：

```java
ManagedChannelImpl channel = NettyChannelBuilder.forAddress("127.0.0.1", 6556).build();
DemoServiceGrpc.DemoServiceBlockingStub stub = DemoServiceGrpc.newBlockingStub(channel);
stub.login(LoginRequest.getDefaultInstance());
```

这三行代码，完成了grpc客户端调用服务器端最重要的三个步骤：

1. 创建连接到远程服务器的 channel
2. 构建使用该channel的客户端stub
3. 调用服务方法，执行RPC调用

## 创建Channel

## 构建Stub

## 执行RPC调用



