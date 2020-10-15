---
date: 2018-09-29T06:54:00+08:00
title: 文档：gRPC名称解析
weight: 10110
menu:
  main:
    parent: "nameresolver-design"
description : "gRPC名称解析"
---

> https://github.com/grpc/grpc/blob/master/doc/naming.md

## 概述

gRPC支持DNS作为默认的名称系统。在不同的部署中，有许多替代的名称系统被使用。我们支持一个足够通用的API，以支持一系列的名称系统和相应的名称语法。各种语言的 gRPC 客户端库将提供一个插件机制，因此可以插入不同名称系统的解析器。

## 详细设计

### 名称语法

用于 gRPC channel 构建的完全限定的自包含名称，使用 RFC 3986中定义的URI语法。

URI schema 指示要使用的解析器插件，如果没有指定schema前缀或schema未知，默认使用dns方案。

URI路径表示要解析的名称。

大多数 gRPC 实现都支持以下URI schema：


- `dns:[//authority/]host[:port]` -- DNS (默认)
  - `host` 是要通过DNS解析的主机名
  - `port` 是要为每个地址返回的端口，如果没有指定，则使用443 （但是某些实现对于非加密channel默认使用80）
  - `authority` 表示要使用的 DNS 服务器，尽管这只被一些实现所支持。在C-core中，默认的DNS解析器不支持这个功能，但基于c-ares的解析器支持以 "IP:port "的形式指定这个功能）。

- `unix:path` or `unix://absolute_path` -- Unix domain sockets (Unix systems only)
  - `path` 表示所需socket的位置。
  - 在第一种形式中，路径可以是相对的，也可以是绝对的；在第二种形式中，路径必须是绝对的（即实际上会有三个斜线，两个在path之前，另一个用来开始绝对路径）。

gRPC C-core实现支持以下方案，但其他语言可能不支持：

- `ipv4:address[:port][,address[:port],...]` -- IPv4 addresses 
  - 可以指定多个逗号分隔的地址，地址形式为`address[:port]`

    - `address` 是使用的 IPv4 地址
    - `port` 是使用的端口。如果没有指定，使用443

- `ipv6:address[:port][,address[:port],...]` -- IPv6 addresses
  - 可以指定多个逗号分隔的地址，地址形式为`address[:port]`
    - `address` 是使用的 IPv6 地址. 要使用 `port` 则地址 `address` 必须用中括号 (`[` and `]`)包起来.  例如:
      `ipv6:[2607:f8b0:400e:c00::ef]:443` or `ipv6:[::]:1234`
    - `port`是使用的端口。如果没有指定，使用443

今后还可以增加其他schema，如 "etcd"。

### Resolver 插件

gRPC客户端类库将使用指定的schema来挑选合适的解析器插件，并将完全限定的名称字符串传递给它。

解析器应该能够联系权威机构（authority）并得到解析，然后将其返回给gRPC客户端类库。返回的内容包括：

- 解析的地址列表（包括IP地址和端口）。每个地址可以有一组与之相关的任意属性（键/值对），这些属性可用于从解析器向负载均衡策略传递信息。

- service config

插件API允许解析器持续观察一个端点，并根据需要返回更新的解析。

