# 资料收集整理

## 网站

- [grpc官网](http://www.grpc.io/)
- [grpc-java](https://github.com/grpc/grpc-java) gRPC Java实现.
- [javadoc](http://www.grpc.io/grpc-java/javadoc/index.html): grpc java 的javadoc地址
- [grpc google groups](https://groups.google.com/forum/#!forum/grpc-io)
- [grpc-ecosystem](https://github.com/grpc-ecosystem)

## 文档

- [grpc-common](http://github.com/grpc/grpc-common) 是官方提供的文档和例子, 但是内容实际是指向下面的grpc.io上的Documentation.
- [Documentation@grpc.io](http://www.grpc.io/docs/) 是grpc.io提供的文档,这个适合入门

Documentation@grpc.io中内容比较重要:

- [overview](http://www.grpc.io/docs/index.html): grpc的overview
- [java教程](http://www.grpc.io/docs/tutorials/basic/java.html) 是官方提供的针对java的教程

> 注：开源中国组织人手翻译这份文档，[gRPC 官方文档中文版](http://doc.oschina.net/grpc?t=58011)

## Demo

- [grpc-android-demo](https://github.com/Lovoo/grpc-android-demo): andriod的demo
- [grpc-streaming-demo](https://github.com/ridha/grpc-streaming-demo): A quick demo of bi-directional streaming RPC's using grpc, go and python
- [yeyincai/grpc-demo](https://github.com/yeyincai/grpc-demo): introduces using grpc about encryption、stream、oneof、interceptor、loadbalance demo http://blog.csdn.net/yeyincai

## 工具

- [grpc-tools](https://github.com/grpc/grpc-tools): Tools useful with gRPC libraries, provided by grpc

## 项目

- [kafka-pixy](https://github.com/mailgun/kafka-pixy): gRPC/REST proxy for Kafka
- [grpc-experiments](https://github.com/grpc/grpc-experiments): Experiments and proposals for gRPC features.
- [grpc-gateway](https://github.com/grpc-ecosystem/grpc-gateway): gRPC to JSON proxy generator
- [LogNet/grpc-spring-boot-starter](https://github.com/LogNet/grpc-spring-boot-starter): Spring Boot starter module for gRPC framework.
- [grpc-opentracing](https://github.com/grpc-ecosystem/grpc-opentracing): OpenTracing is a set of consistent, expressive, vendor-neutral APIs for distributed tracing and context propagation

## 周边项目

- [grpc-gateway](https://github.com/gengo/grpc-gateway)： 是一个基于go语言的项目.

    > grpc-gateway是protoc的插件. 它读取gRPC 服务定义, 然后生成一个反向代理服务器, 将RESTful JSON API转为gRPC.
    >
    > 用于帮助为API同时提供gRPC 和 RESTful接口.

    这个工具似乎不错,对于某些需要提供restul接口场合可以快速的在grpc接口上转换出来.

- [grpc-docker-library](https://github.com/grpc/grpc-docker-library): 包含官方gRPC Docker镜像的Git仓库
- [grpc-spring-boot-starter](https://github.com/yidongnan/grpc-spring-boot-starter): 介绍 [gRPC Spring Boot Starter - SprintBoot 的 gRPC 模块](https://www.v2ex.com/t/343538)



