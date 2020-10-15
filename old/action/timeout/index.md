# 超时

在 gRPC 中没有找到传统的超时设置，只看到在 stub 上有 deadline 的设置。但是这个是设置整个 stub 的 deadline，而不是单个请求。

后来通过一个 [deadline 的 issue](https://github.com/grpc/grpc-java/issues/1495) 了解到，其实可以这样来实现针对每次 RPC 请求的超时设置：

```java
for (int i=0; i<100; i++) {
    blockingStub.withDeadlineAfter(3, TimeUnit.SECONDS).doSomething();
}
```

这里的 .withDeadlineAfter() 会在原有的 stub 基础上新建一个 stub，然后如果我们为每次 RPC 请求都单独创建一个有设置 deadline 的 stub，就可以实现所谓单个 RPC 请求的 timeout 设置。

但是代价感觉有点高，每次RPC都是一个新的 stub，虽然底层肯定是共享信息。

TBD： 实际跑个测试试试性能，如果没有明显下降，就可以考虑采用这种方法。

