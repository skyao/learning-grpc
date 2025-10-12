---
title: "[译]gRPC 中的 xDS 功能"
linkTitle: "[译]gRPC 中的 xDS 功能"
weight: 30
date: 2023-01-29
description: >
  gRPC 中的 xDS 功能
---



> 原文：[GRPC Core: xDS Features in gRPC](https://grpc.github.io/grpc/core/md_doc_grpc_xds_features.html)



本文档列出了各种 gRPC 语言实现和版本所支持的 xDS 功能。

请注意，gRPC 客户端将简单地忽略它不支持的功能配置。gRPC 客户端不会生成日志来表明某些配置被忽略了。生成日志并保持更新是不切实际的，因为 xDS 有大量的 API 是 gRPC 不支持的，而且这些 API 也在不断发展。在一个 xDS 字段对应的功能被支持，但为该字段配置的值不被支持的情况下，gRPC 客户端将 NACK 这样的配置。我们建议阅读第一个关于 gRPC 中 xDS 支持的 gRFC，以了解设计理念。

不是所有的集群负载平衡策略都被支持。gRPC 客户端将 NACK 包含不支持集群负载平衡策略的配置。这将导致所有集群配置被客户端拒绝，因为 xDS 协议目前要求拒绝给定响应中的所有资源，而不是能够只拒绝响应中的个别资源。由于这一限制，在为服务配置所需的集群负载平衡策略之前，你必须确保所有客户端都支持该策略。例如，如果你把 ROUND_ROBIN 策略改为 RING_HASH，你必须确保所有客户端都升级到支持 RING_HASH 的版本。

EDS 策略将不支持 超额配置，这与 Envoy 不同。Envoy 在本地加权负载平衡和优先级故障转移中都考虑到了超额配置，但 gRPC 假设 xDS 服务器会在需要这种优雅故障转移时更新它来重定向流量。gRPC 将向 xDS 服务器发送 envoy.lb.does_not_support_overprovisioning 客户特征，告诉 xDS 服务器它不会执行优雅故障转移；xDS 服务器实现可以用它来决定自己是否执行优雅故障转移。

EDS 策略将不支持每端点（per-endpoint）统计；它将只报告每地点（per-locality）的统计。

如果一个 lb_endpoint 的健康状态不是 HEALTHY 或 UNKNOWN，则会被忽略。可选的 load_balancing_weight 总是被忽略。

最初，只支持 google_default 通道认证与 xDS 服务器进行认证。

下表中未列出的 gRPC 语言实现不支持 xDS 功能：

| 功能                                                         | gRFCs                                                        | [C++, Python, Ruby, PHP](https://github.com/grpc/grpc/releases) | [Java](https://github.com/grpc/grpc-java/releases) | [Go](https://github.com/grpc/grpc-go/releases) | [Node](https://github.com/grpc/grpc-node/releases) |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | -------------------------------------------------- | ---------------------------------------------- | -------------------------------------------------- |
| **gRPC 客户端通道中的 xDS 基础设置:** <br />- LDS->RDS->CDS->EDS flow<br />- ADS stream | [A27](https://github.com/grpc/proposal/blob/master/A27-xds-global-load-balancing.md) | v1.30.0                                                      | v1.30.0                                            | v1.30.0                                        | v1.2.0                                             |
| **负载均衡:** <br />- [Virtual host](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#config-route-v3-virtualhost) domains matching<br />- Only default path ("" or "/") matching<br />- Priority-based weighted round-robin locality picking<br />- Round-robin endpoint picking within locality<br />- [Cluster](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#config-route-v3-routeaction) route action<br />- Client-side Load reporting via [LRS](https://github.com/envoyproxy/data-plane-api/blob/master/envoy/service/load_stats/v3/lrs.proto) | [A27](https://github.com/grpc/proposal/blob/master/A27-xds-global-load-balancing.md) | v1.30.0                                                      | v1.30.0                                            | v1.30.0                                        | v1.2.0                                             |
| Request matching based on:<br /><br />- [Path](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#config-route-v3-routematch) (prefix, full path and safe regex) <br />      - [case_sensitive](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-routematch) must be true else config is NACKed<br />- [Headers](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-headermatcher)<br /><br />Request routing to multiple clusters based on [weights](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#config-route-v3-weightedcluster) | [A28](https://github.com/grpc/proposal/blob/master/A28-xds-traffic-splitting-and-routing.md) | v1.31.0                                                      | v1.31.0                                            | v1.31.0                                        | v1.3.0                                             |
| Case insensitive prefix/full path matching:[case_sensitive](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-routematch) can be true or false |                                                              | v1.34.0                                                      | v1.34.0                                            | v1.34.0                                        | v1.3.0                                             |
| Support for [xDS v3 APIs](https://www.envoyproxy.io/docs/envoy/latest/api-v3/api) | [A30](https://github.com/grpc/proposal/blob/master/A30-xds-v3.md) | v1.36.0                                                      | v1.36.0                                            | v1.36.0                                        | v1.4.0                                             |
| Support for [xDS v2 APIs](https://www.envoyproxy.io/docs/envoy/latest/api/api_supported_versions) | [A27](https://github.com/grpc/proposal/blob/master/A30-xds-v3.md#details-of-the-v2-to-v3-transition) | < v1.51.0                                                    | < v1.53.0                                          | TBA                                            | < v1.8.0                                           |
| [Maximum Stream Duration](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#config-route-v3-routeaction-maxstreamduration):Only max_stream_duration is supported. | [A31](https://github.com/grpc/proposal/blob/master/A31-xds-timeout-support-and-config-selector.md) | v1.37.1                                                      | v1.37.1                                            | v1.37.0                                        | v1.4.0                                             |
| [Circuit Breaking](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/cluster/v3/circuit_breaker.proto):Only max_requests is supported. | [A32](https://github.com/grpc/proposal/blob/master/A32-xds-circuit-breaking.md) | v1.37.1 (N/A for PHP)                                        | v1.37.1                                            | v1.37.0                                        | v1.4.0                                             |
| [Fault Injection](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/fault/v3/fault.proto): Only the following fields are supported:delayabortmax_active_faultsheaders | [A33](https://github.com/grpc/proposal/blob/master/A33-Fault-Injection.md) | v1.37.1                                                      | v1.37.1                                            | v1.37.0                                        | v1.4.0                                             |
| [Client Status Discovery Service](https://github.com/envoyproxy/envoy/blob/main/api/envoy/service/status/v3/csds.proto) | [A40](https://github.com/grpc/proposal/blob/master/A40-csds-support.md) | v1.37.1 (C++) v1.38.0 (Python)                               | v1.37.1                                            | v1.37.0                                        | v1.5.0                                             |
| [Ring hash](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/load_balancing/load_balancers#ring-hash) load balancing policy: Only the following [policy specifiers](https://github.com/envoyproxy/envoy/blob/2443032526cf6e50d63d35770df9473dd0460fc0/api/envoy/config/route/v3/route_components.proto#L706) are supported:headerfilter_state with key `io.grpc.channel_id`Only [`XX_HASH`](https://github.com/envoyproxy/envoy/blob/2443032526cf6e50d63d35770df9473dd0460fc0/api/envoy/config/cluster/v3/cluster.proto#L383) function is supported. | [A42](https://github.com/grpc/proposal/blob/master/A42-xds-ring-hash-lb-policy.md) | v1.40.0 (C++ and Python)                                     | v1.40.1                                            | 1.41.0                                         |                                                    |
| [Retry](https://www.envoyproxy.io/docs/envoy/latest/api-v3/config/route/v3/route_components.proto#envoy-v3-api-msg-config-route-v3-retrypolicy): Only the following fields are supported:retry_on for the following conditions: cancelled, deadline-exceeded, internal, resource-exhausted, and unavailable.num_retriesretry_back_off | [A44](https://github.com/grpc/proposal/blob/master/A44-xds-retry.md) | v1.40.0 (C++ and Python)                                     | v1.40.1                                            | 1.41.0                                         |                                                    |
| [Security](https://www.envoyproxy.io/docs/envoy/latest/configuration/security/security): Uses [certificate providers](https://github.com/grpc/proposal/blob/master/A29-xds-tls-security.md#certificate-provider-plugin-framework) instead of SDS | [A29](https://github.com/grpc/proposal/blob/master/A29-xds-tls-security.md) | v1.41.0 (C++ and Python)                                     | v1.41.0                                            | 1.41.0                                         |                                                    |
| [Authorization (RBAC)](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/filters/http/rbac/v3/rbac.proto): `LOG` action has no effectCEL unsupported and rejected | [A41](https://github.com/grpc/proposal/blob/master/A41-xds-rbac.md) | v1.51.0 (C++ and Python)                                     | v1.42.0                                            | 1.42.0                                         |                                                    |

[Outlier Detection](https://www.envoyproxy.io/docs/envoy/latest/intro/arch_overview/upstream/outlier):
Only the following detection types are supported:

- Success Rate
- Failure Percentage



备注：

截止2023年6月，最新的版本是：

- Grpc-java: 1.55.1
- grpc-go： 1.54.1
- grpc-node：1.8.15
- grpc （b based，C++, Python, Ruby, Objective-C, PHP, C#）： 1.55.1
- grpc官方没有rust的sdk！

