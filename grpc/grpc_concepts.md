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

# RPC life cycle

## Unary/一元 RPC

最简单的RPC类型, 客户端发送单一请求然后获取单衣应答.

- 一旦客户端调用stub上的方法, 服务器就被通知RPC已经被调用了, 还有客户端这次调用的metadata, 方法名, 以及指定的deadlin(如果有).
- 服务器要不直接发送会它自己的初始化metadata(必须在任何应答之前发送), 要不等待客户端的请求消息 - 具体怎么做取决于应用.
- 一旦服务器拿到了客户端的请求信息, 它会竭尽所能的创建应答. 应答随即和状态详情(状态码和可选状态消息)以及可选 trailing metadata返回(如果成功)到客户端
- 如果状态OK, 客户端就获取应答, 完成客户端的调用

## Server streaming RPC

Server streaming RPC 在收到客户端请求消息后发送会一个响应的stream. 在发送完所有的应答之后, 服务器端的状态详情(状态码和可选状态消息)和可选trailing metadata会发送来完成服务器处理. 客户端一旦收到所有的服务器的应答就完成处理.

## Client streaming RPC

Client streaming RPC中, 客户端发送一个请求的stream到服务器而不是一个单一的请求. 服务器发送回一个单一的应答, 通常但不是必须在它接收到所有的客户端请求后, 同样带有状态详情和可选 trailing metadata.

## Bidirectional streaming RPC

在Bidirectional streaming RPC中, 调用的发起还是由客户端调用方法和服务器接收客户端metadata, 方法名和deadline开始. 服务器还是可以选择发送回它的初始化metadata还是等待客户端开始发送请求.

下一步如何取决于应用, 因为客户端和服务器端可以以任意顺序读写 - stream操作是完全独立的. 因此, 例如, 服务器可以等待直到它接收完所有客户端的消息再发送应答, 或者服务器端和客户端可以"ping-pong": 服务器接收一个请求,发送一个应答, 然后客户端基于应答再发送另一个请求, 以此类推.

# Deadline

gRPC容许客户端在调用一个远程方法时指定一个deadline的值. 这会指定客户端在RPC以错误 DEADLINE_EXCEEDED 结束前等待服务器应答的时间长度. 在服务器那边, 服务器会查询deadline来看特定的方法是否已经超时, 或者还有多长时间剩余来完成这个方法.

deadline如何指定和从哪个语言到哪个语言有关.

# 取消RPC

客户端和服务器端都可以在任何时间取消一个RPC调用. 取消操作会立即终止RPC因此后面不会再有工作. 取消不是"undo": 在取消之前已经做的改变不会回滚. 当然, 通过同步方法调用的RPC不能被取消, 因为直到
被终结否则程序控制不会返回到应用.

# metadata

metadata 是关于一个特定RPC调用的信息(例如 认证信息), 以键值对列表的形式, key是string而值也通常是string(但是可以是二进制数据). detadata对gRPC自身是不透明的 - 它让客户端可以提供和调用关联的信息到服务器, 反之亦然.


# Channels

gRPC Channel 提供一个到特定主机和端口的gRPC服务器的连接, 在创建客户端stub时使用. 客户端可以指定channel 参数来改变gRPC的默认行为, 比如切换消息压缩的开关. Channel有状态, 包括connected 和 idel.












