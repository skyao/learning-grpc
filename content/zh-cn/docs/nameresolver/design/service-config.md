---
date: 2018-09-29T06:54:00+08:00
title: 文档：gRPC服务配置
weight: 10120
description : "gRPC服务配置"
---

> https://github.com/grpc/grpc/blob/master/doc/service_config.md

### 目标

服务配置是一种机制，它允许服务所有者发布参数，让其服务的所有客户端自动使用。

### 格式

服务配置的格式由 [`grpc.service_config.ServiceConfig` protocol buffer
message](https://github.com/grpc/grpc-proto/blob/master/grpc/service_config/service_config.proto)  定义。请注意，随着新功能的引入，未来可能会添加新的字段。

### 架构

服务配置与服务器名相关联。名称解析器插件在被要求解析某个服务器名称时，会同时返回解析的地址和服务配置。

名称解析器以 JSON 形式将服务配置返回给 gRPC 客户端。各个解析器实现决定服务配置的存储位置和格式。如果解析器实现以 protobuf 形式获取服务配置，则必须使用正常的 protobuf 到 JSON 的转换规则将其转换为 JSON。另外，解析器实现也可以以 JSON 形式获取服务配置，在这种情况下，它可以直接返回服务配置。

有关 DNS 解析器插件如何支持服务配置的详情，请参见 [gRFC A2: Service Config via
DNS](https://github.com/grpc/proposal/blob/master/A2-service-configs-in-dns.md).

### 例子

下面是一个protobuf形式的服务配置示例：

```json
{
  // 使用 round_robin 负载均衡策略
  load_balancing_config: { round_robin: {} }
  // 这个方法配置适用于 "foo/bar" 方法和 service "baz"的所有方法
  method_config: {
    name: {
      service: "foo"
      method: "bar"
    }
    name: {
      service: "baz"
    }
    // 匹配方法的默认超时
    timeout: {
      seconds: 1
      nanos: 1
    }
  }
}
```

下面是同样的JSON形式的服务配置示例：

```
{
  "loadBalancingConfig": [ { "round_robin": {} } ],
  "methodConfig": [
    {
      "name": [
        { "service": "foo", "method": "bar" },
        { "service": "baz" }
      ],
      "timeout": "1.0000000001s"
    }
  ]
}
```

### API

服务配置在以下API中使用：

- 在 resolver API中，用于解析器插件，以便将服务配置返回给gRPC客户端
- 在 gRPC 客户端 API 中，用户可以查询 channel 以获取与通道相关的服务配置（用于调试）。
- 在 gRPC 客户端 API 中，用户可以显式地设置服务配置。这可以用来在单元测试中设置配置。它还可以用来设置默认配置，如果解析器插件没有返回服务配置，就会使用该配置。
