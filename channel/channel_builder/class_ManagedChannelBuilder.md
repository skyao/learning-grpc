类ManagedChannelBuilder
======================

用于创建 ManagedChannel 实例的 builder.

# 类定义

```java
package io.grpc;
public abstract class ManagedChannelBuilder<T extends ManagedChannelBuilder<T>> {
}
```

T 是这个builder的具体类型。

# 类方法

## 静态工厂方法

有两个静态方法用来实例。

ManagedChannelProvider 的使用方式和设计实现，不在这里展开，在下一节 [Channel Provider](../channel_provider/index.md) 中再详细介绍。

### forAddress()

```java
public static ManagedChannelBuilder<?> forAddress(String name, int port) {
	return ManagedChannelProvider.provider().builderForAddress(name, port);
}
```

### forTarget()

```java
@ExperimentalApi
public static ManagedChannelBuilder<?> forTarget(String target) {
	return ManagedChannelProvider.provider().builderForTarget(target);
}
```

用target字符串创建一个channel， 可以是一个有效的命名解析器兼容(NameResolver-compliant)的URI，或者是一个 authority 字符串。

> 注： authority 是URI中的术语， URI的标准组成部分如 `[scheme:][//authority][path][?query][#fragment]`， authority代表URI中的 `[userinfo@]host[:port]`，包括host(或者ip)和可选的port和userinfo。

命名解析器兼容(NameResolver-compliant)URI 是被作为URI定义的抽象层次(absolute hierarchical)URI。实例的URI：

- "dns:///foo.googleapis.com:8080"
- "dns:///foo.googleapis.com"
- "dns:///%5B2001:db8:85a3:8d3:1319:8a2e:370:7348%5D:443"
- "dns://8.8.8.8/foo.googleapis.com:8080"
- "dns://8.8.8.8/foo.googleapis.com"
- "zookeeper://zk.example.com:9900/example_service"

authority字符串将被转换为一个命名解析器兼容的URI， 使用"dns"作为scheme，没有authority，而且用合适转义之后的原始authority字符串作为它的path：

- "localhost"
- "127.0.0.1"
- "localhost:8080"
- "foo.googleapis.com:8080"
- "127.0.0.1:8080"
- "[2001:db8:85a3:8d3:1319:8a2e:370:7348]"
- "[2001:db8:85a3:8d3:1319:8a2e:370:7348]:443"

> 注: 以上内容来自javadoc上该方法的注释说明。

## 实例方法

### directExecutor()

```java
public abstract T directExecutor();
```

直接在传输的线程中执行应用代码。

取决于底层传输，使用一个 direct executor 可能引发重大的性能提升。当然，它也要求应用在任何情况下不阻塞。

调用这个方法在语义上等同于调用 executor(Executor) 并传递一个 direct executor。但是，这个方式更合适因为它可以容许传输层执行特别的优化。

> 注： 考虑实际测试一下，看是否真的可以得到重大的性能提升。

### executor()

```java
public abstract T executor(Executor executor);
```

提供一个自定义executor。

这是一个可选参数。如果在channel构建时没有提供executor，builder将使用一个静态缓存的线程池。

channel不会承担给定 executor 的所有者职责。当需要时，关闭 executor 是调用者的责任。

> 注： 这意味着我们作为调用者需要自己管理好 executor。

### intercept()

```java
public abstract T intercept(List<ClientInterceptor> interceptors);
```

添加拦截器，被channel执行它的实际工作前被调用。功能上等同于使用 ClientInterceptors.intercept(Channel, List)， 但是依然可以访问原始的ManagedChannel。

```java
public abstract T intercept(ClientInterceptor... interceptors);
```

添加拦截器，被channel执行它的实际工作前被调用。功能上等同于使用 ClientInterceptors.intercept(Channel, ClientInterceptor...)， 但是依然可以访问原始的ManagedChannel。

### userAgent()

```java
public abstract T userAgent(String userAgent);
```

为应用提供一个定制的 User-Agent 。

这是一个可选参数。如果提供，给定的agent将使用grpc User-Agent作为前缀.

> 注： 原文 the given agent will be prepended by the grpc User-Agent. 硬是没有看懂到底哪个作为前缀？ 等后面查代码。

### overrideAuthority()

```java
@ExperimentalApi(value="primarily for testing")
public abstract T overrideAuthority(String authority)
```

覆盖和TLS和HTTP 虚拟主机服务一起使用的authority。它不会改变实际连接到的主机。通常是以host:port的形式。

应该仅用于测试。

### usePlaintext()

```java
@ExperimentalApi("primarily for testing")
public abstract T usePlaintext(boolean skipNegotiation);
```

使用 plaintext 连接到服务器。默认将使用加密连接机制如 TLS 。

应该仅用于测试或者用于那些API使用或者数据交换并不敏感的API。

参数 skipNegotiation true，如果预先知道终端支持plaintext; false 如果 plaintext 的使用必须协商。

### nameResolverFactory()

```java
@ExperimentalApi
public abstract T nameResolverFactory(NameResolver.Factory resolverFactory);
```

为channel提供一个定制的 NameResolver.Factory。如果这个方法没有被调用，builder将在全局解析器注册(global resolver registry)中为提供的目标寻找一个工厂。

### loadBalancerFactory()

```java
  @ExperimentalApi
  public abstract T loadBalancerFactory(LoadBalancer.Factory loadBalancerFactory);
```

为channel提供一个定制的 LoadBalancer.Factory 。

如果这个方法没有被调用，builder将为这个 channel 使用 SimpleLoadBalancerFactory 。

### decompressorRegistry()

```java
@ExperimentalApi
public abstract T decompressorRegistry(DecompressorRegistry registry);
```

设置用于在channen中使用的解压缩注册器。这是一个高级API调用，不应该被使用，除非你正在使用定制化的消息编码。默认支持的解压缩在 DecompressorRegistry.getDefaultInstance 中。

### compressorRegistry()

```java
@ExperimentalApi
public abstract T compressorRegistry(CompressorRegistry registry);
```

设置用于在channen中使用的压缩注册器。这是一个高级API调用，不应该被使用，除非你正在使用定制化的消息编码。默认支持的解压缩在 DecompressorRegistry.getDefaultInstance 中。

### idleTimeout()

```java
@ExperimentalApi
public abstract T idleTimeout(long value, TimeUnit unit);
```

设置在进入空闲模式前的没有RPC的期限。

在空闲模式中， channel会关闭所有连接， NameResolver 和 LoadBalancer 。新的RPC将把channel带出空闲模式。channel以空闲模式开始。

默认，在离开初始化空闲模式 channel 将从不会进入空闲模式

### build()

使用给定参数来构建一个channel

```java
@ExperimentalApi
public abstract ManagedChannel build();
```


