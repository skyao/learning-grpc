---
date: 2018-09-29T06:54:00+08:00
title: 提案：service configs via dns
weight: 10130
description : "通过dns进行服务配置"
---

> https://github.com/grpc/proposal/blob/master/A2-service-configs-in-dns.md

## 摘要

本文档提出了一种在DNS中编码gRPC服务配置数据的机制，供开源世界使用。

## 背景

服务配置机制最初是为Google内部使用而设计的。然而，除了一部分之外，所有原始设计在开源世界中都能正常工作。这一部分就是服务配置数据在DNS中如何编码的规范。这个提案填补了这个缺失的部分。

## 提案

这个提案有两个部分。第一部分是增加一些JSON包装，用于控制服务配置的更改如何进行金丝雀测试。第二部分是描述服务配置如何在DNS中进行编码。

### Canarying Changes

### 金丝雀变更

当部署对服务配置的变更时，通过缓慢增加看到新版本的客户端数量，能够对更改进行金丝雀测试以避免大范围的中断，这是非常有用的。为此，可以按顺序列出多个服务配置的选择，以及决定特定客户机将选择哪个的标准：

```json
// A list of one or more service config choices.
// The first matching entry wins.
[
  {
    // Criteria used to select this choice.
    // If a field is absent or empty, it matches all clients.
    // All fields must match a client for this choice to be selected.
    // If any unexpected field name is present in this object, the entire
    // config is considered invalid.
    //
    // Client language(s): a list of strings (e.g., "c++", "java", "go",
    // "python", etc).  Each string is case insensitive.
    "clientLanguage": [string],
    // Percentage: integer from 0 to 100 indicating the percentage of
    // clients that should use this choice.  If present, the number must
    // match the regular expression `^0|[0-9]|[1-9][0-9]|100$`
    // All other numbers are considered invalid.
    "percentage": number,
    // Client hostname(s): a list of strings.  Each name is case 
    // sensitive and must be an exact match of the hostname according to
    // the system.
    "clientHostname": [string],

    // The service config data object for clients that match the above
    // criteria.  (The format for this object is defined in
    // https://github.com/grpc/grpc/blob/master/doc/service_config.md.)
    // If this field is not an object, or is missing, or is otherwise 
    // invalid, the entire config is considered invalid.
    "serviceConfig": object
  }
]
```

如果服务配置选择不能被解析，或者在其他方面语义上无效，整个配置必须按照   [Service Config Error Handling](https://github.com/grpc/proposal/blob/master/A21-service-config-error-handling.md) 丢弃。


### 在DNS TXT记录中编码

在DNS中，服务配置数据(以上一节中记录的形式)将通过RFC-1464中描述的机制，使用属性名 grpc_config 在TXT记录中进行编码。属性值将是一个包含服务配置选择的JSON列表。TXT记录将是一个与gRPC服务器名称相同的DNS名称，但前缀为 `_grpc_config.` ......

例如，这里是服务器 myserver 的 TXT 记录示例：

```json
_grpc_config.myserver  3600  TXT "grpc_config=[{\"serviceConfig\":{\"loadBalancingPolicy\":\"round_robin\",\"methodConfig\":[{\"name\":[{\"service\":\"MyService\",\"method\":\"Foo\"}],\"waitForReady\":true}]}}]"
```

请注意，根据RFC-1035第3.3节的规定，TXT记录每个字符串限制为255字节。然而，可以有多个字符串，它们将被连接在一起，如RFC-4408第3.1.3节所述。总的DNS响应不能超过65535字节。(更多讨论请参见下面的 "未解决的问题 "部分。)

需要注意的是，由于TXT记录必须是ASCII码，这也限制了服务配置的内容也是ASCII码（如服务和方法名称、负载均衡策略名称等）。

## 理由

服务配置被设计为作为名称解析的一部分而返回，所以在DNS中进行编码是最合理的。当然，使用 DNS 以外的其他命名系统的网站可以用自己的机制实现自己的解析器，对服务配置数据进行编码。

当在DNS中对服务配置进行编码时，TXT记录是 "显而易见 "的选择，因为服务配置实际上是与DNS名称相关联的附加元数据。

我们在DNS条目中使用 `_grpc_config` 前缀，允许为主记录为CNAME记录的服务指定服务配置，因为DNS不允许为包含CNAME记录的同一名称指定任何其他记录。

## 实现

在C-core中，作为c-ares DNS解析器的一部分，已经完成了实施。我们目前正在努力使c-ares解析器成为C-core的默认DNS解析器。这需要诸如Windows和Node的支持，以及增加地址排序。

## 未解决的问题（如果适用）

DNS TXT记录确实有一些限制，这里需要考虑到。尤其是

- 如果DNS响应超过512字节 就会从UDP退回到TCP，这就增加了开销
- DNS的总响应不能超过65535字节。
- 目前还不清楚各个DNS实现是否会允许接近65535字节，尽管规范中说应该允许。

请反馈这些考虑因素是否会成为本设计的重大缺点（在这种情况下，很可能要改变设计）。