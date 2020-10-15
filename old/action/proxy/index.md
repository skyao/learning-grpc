# 代理

考虑一下如果有需要做gRPC的代理，该如何进行。

## nghttpx

https://nghttp2.org/documentation/nghttpx.1.html

用于 HTTP/2, HTTP/1 和 SPDY 的反向代理。

结论：看介绍是支持了，有待进一步确认。

## GCLB

GCLB 指 Google Global Load Balancer, 地址 https://cloud.google.com/compute/docs/load-balancing/http/

这里有一个详细的讨论 [Google Global Load Balancer support for gRPC?](https://groups.google.com/forum/?utm_medium=email&utm_source=footer#!msg/grpc-io/bfURoNLojHo/GwEFOXumAQAJ)

结论：目前 GCLB 只能支持到 Layer-3 的负载均衡，Layer-7 还在开发中，预计是2016年底或者2017年才能发布。

## grpc-proxy

gRPC proxy is a Go reverse proxy that allows for rich routing of gRPC calls with minimum overhead.

https://github.com/mwitkow/grpc-proxy

结论：目前还是 alpha 状态