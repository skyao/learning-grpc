gRPC概念
====

注: 官网文档[gRPC Concepts](http://www.grpc.io/docs/guides/concepts.html)的读书笔记.

# 概述

## 服务定义

和很多RPC系统一样, gRPC也是基于这样的想法: 定义服务, 指定可以远程调用的方法, 和方法的参数和返回类型.

gRPC容许定义四种服务方法:

1. 一元RPC调用, 客户端发送一个单一的请求到服务器然后得到一个单一相应, 就像普通函数调用.

    ```java
    rpc SayHello(HelloRequest) returns (HelloResponse){
    }
    ```

2. 服务端stream RPC, 客户端发送一个请求到服务器然后得到一个流(stream), 可以读取消息队列. 客户端从返回的流中读取直到没有更多消息.

    ```java
    rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse){
    }
    ```

3. 客户端stream RPC, 客户端写消息序列并发送到服务器, 使用提供的stream. 一旦客户端完成消息写入, 它等待服务器读取消息并返回响应.
    ```java
    rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
    }
    ```

4. 双向 streaming RPC, 两端都使用读写steam来发送消息序列. 两个流独立操作, 因此客户端和服务器端可以一他们喜欢的任意顺序读写: 例如, 服务器可以在写响应前等待所有客户端消息, 或者交替读一个消息然后写一个消息, 或者一些其他读写的结合. 每个流的消息顺序是有保证的.

    ```java
    rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){
    }
    ```




