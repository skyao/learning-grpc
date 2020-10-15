# google grpc 介绍
================

grpc是google最新发布(2015年2月底)的开源rpc框架。

按照google的说法，grpc是：

**A high performance, open source, general RPC framework that puts mobile and HTTP/2 first.**

一个高性能，开源，将移动和HTTP/2放在首位的通用的RPC框架.

# 特性

## HTTP/2

构建于HTTP/2标准，带来很多功能，如：

- bidirectional streaming
- flow control
- header compression
- multiplexing requests over a single TCP connection

## mobile

这些特性在移动设备上节约电池使用时间和数据使用，加速服务和运行在云上的web应用。