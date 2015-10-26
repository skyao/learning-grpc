Google grpc 项目相关的信息整理:

# 网站

## 官方网站

- [grpc官网](http://www.grpc.io/)

## 源代码

源代码托管于github：

- [grpc](https://github.com/grpc/grpc) The C based gRPC (C++, Node.js, Python, Ruby, Objective-C, PHP, C#)
- [grpc-java](https://github.com/grpc/grpc-java) The Java gRPC implementation. HTTP/2 based RPC
- [grpc-go](https://github.com/grpc/grpc-go) The Go language implementation of gRPC. HTTP/2 based RPC

## 文档和例子

- [grpc-common](http://github.com/grpc/grpc-common) 是官方提供的文档和例子, 但是内容实际是指向下面的grpc.io上的Documentation.
- [Documentation@grpc.io](http://www.grpc.io/docs/tutorials/basic/java.html) 是grpc.io提供的文档,这个适合入门

Documentation@grpc.io中内容比较重要:

- [overview](http://www.grpc.io/docs/index.html): grpc的overview
- [java教程](http://www.grpc.io/docs/tutorials/basic/java.html) 是官方提供的针对java的教程

另外github上有一些其他例子:

- [grpc-android-demo](https://github.com/Lovoo/grpc-android-demo): andriod的demo

## 工具

- [grpc-tools](https://github.com/grpc/grpc-tools): Tools useful with gRPC libraries, provided by grpc

# 周边项目

## grpc-gateway

[grpc-gateway](https://github.com/gengo/grpc-gateway) 是一个基于go语言的项目.

> grpc-gateway是protoc的插件. 它读取gRPC 服务定义, 然后生成一个反向代理服务器, 将RESTful JSON API转为gRPC.
>
> 用于帮助为API同时提供gRPC 和 RESTful接口.

这个工具似乎不错,对于某些需要提供restul接口场合可以快速的在grpc接口上转换出来.

## grpc-docker-library

[grpc-docker-library](https://github.com/grpc/grpc-docker-library):  Git repo containing the official Docker images for grpc

注: 比较奇怪的是这里有各个语言的dockfile,比如c/go/c#/php/python/ruby等,就是没有java...... OK, 搞情况了,对于grpc的支持,java不需要特别安装或者设置,只要有jdk就足以.





