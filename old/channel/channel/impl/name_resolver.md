# Name Resolver

## 构建 Name Resolver

ManagedChannelImpl中和 Name Resolver 相关的属性：

```java
// 匹配这个正则表达式意味着 target 字符串是一个 URI target，或者至少打算成为一个
// URI target 必须是一个absolute hierarchical URI
// 来自 RFC 2396: scheme = alpha *( alpha | digit | "+" | "-" | "." )
static final Pattern URI_PATTERN = Pattern.compile("[a-zA-Z][a-zA-Z0-9+.-]*:/.*");

private final String target;
private final NameResolver.Factory nameResolverFactory;
private final Attributes nameResolverParams;

// 绝不为空。必须在lock之下修改。
// 这里不能是final，因为后面会重新设值，但是绝不会设置为null
private NameResolver nameResolver;
```

ManagedChannelImpl()的构造函数，传入target/nameResolverFactory/nameResolverParams：

```java
ManagedChannelImpl(String target, ......
      NameResolver.Factory nameResolverFactory, Attributes nameResolverParams,......) {
    this.target = checkNotNull(target, "target");
    this.nameResolverFactory = checkNotNull(nameResolverFactory, "nameResolverFactory");
    this.nameResolverParams = checkNotNull(nameResolverParams, "nameResolverParams");
    this.nameResolver = getNameResolver(target, nameResolverFactory, nameResolverParams);
    ......
  }
```

getNameResolver() 方法根据这三个参数构建 NameResolver：

```java
static NameResolver getNameResolver(String target, NameResolver.Factory nameResolverFactory, Attributes nameResolverParams) {
    //查找NameResolver。尝试使用 target 字符串作为 URI。如果失败，尝试添加前缀
    // "dns:///".
    URI targetUri = null;
    StringBuilder uriSyntaxErrors = new StringBuilder();
    try {
      targetUri = new URI(target);
      // 对于 "localhost:8080" 这将会导致 newNameResolver() 方法返回null
      // 因为 "localhost" 被作为 scheme 解析。
      // 将会转入下一个分支并尝试 "dns:///localhost:8080"
    } catch (URISyntaxException e) {
      // 可以发生在类似 "[::1]:1234" or 127.0.0.1:1234 这样的ip地址
      uriSyntaxErrors.append(e.getMessage());
    }
    if (targetUri != null) {
      // 如果 target 是有效的URI
      NameResolver resolver = nameResolverFactory.newNameResolver(targetUri, nameResolverParams);
      if (resolver != null) {
        return resolver;
      }
      // "foo.googleapis.com:8080" 导致 resolver 为null，因为 "foo.googleapis.com" 是一个没有映射的scheme。简单失败并将尝试"dns:///foo.googleapis.com:8080"
    }

    // 如果走到这里，表明 targetUri 不能使用
    if (!URI_PATTERN.matcher(target).matches()) {
      // 如果格式都不匹配，说明看上去不像是一个 URI target。可能是 authority 字符串。
      // 尝试从factory中获取默认 scheme
      try {
        targetUri = new URI(nameResolverFactory.getDefaultScheme(), "", "/" + target, null);
      } catch (URISyntaxException e) {
        // Should not be possible.
        throw new IllegalArgumentException(e);
      }
      if (targetUri != null) {
        // 尝试加了默认 scheme 之后的URI
        NameResolver resolver = nameResolverFactory.newNameResolver(targetUri, nameResolverParams);
        if (resolver != null) {
          return resolver;
        }
      }
    }
    // 最后如果还是不成，只能报错了
    throw new IllegalArgumentException(String.format(
        "cannot find a NameResolver for %s%s",
        target, uriSyntaxErrors.length() > 0 ? " (" + uriSyntaxErrors + ")" : ""));
}
```

构建失败会直接抛异常退出。

## Name Resolver 的使用

在 TransportSet.Callback 中被调用：

```java
final TransportManager<ClientTransport> tm = new TransportManager<ClientTransport>() {
    public ClientTransport getTransport(final EquivalentAddressGroup addressGroup) {
      ......
      ts = new TransportSet(......,
        new TransportSet.Callback() {
            public void onAllAddressesFailed() {
              // 所有地址失败时刷新
              nameResolver.refresh();
            }

            public void onConnectionClosedByServer(Status status) {
              // 服务器端关闭连接时刷新
              nameResolver.refresh();
            }
		......
```

### NameResolver.Listener

NameResolver通过 NameResolver.Listener 来通知变化，这里的实现是类NameResolverListenerImpl：

```java
private class NameResolverListenerImpl implements NameResolver.Listener {
    final LoadBalancer<ClientTransport> balancer;

    NameResolverListenerImpl(LoadBalancer<ClientTransport> balancer) {
      this.balancer = balancer;
    }

    @Override
    public void onUpdate(List< ? extends List<ResolvedServerInfo>> servers, Attributes config) {
      if (serversAreEmpty(servers)) {
        // 如果更新的服务列表为空，表明没有可用的服务器了
        onError(Status.UNAVAILABLE.withDescription("NameResolver returned an empty list"));
      } else {
        // 如果更新的服务列表不为空，通知balancer
        try {
          balancer.handleResolvedAddresses(servers, config);
        } catch (Throwable e) {
          // 这必然是一个bug！将异常推回给 LoadBalancer ，希望LoadBalancer可以提升给应用
          balancer.handleNameResolutionError(Status.INTERNAL.withCause(e)
              .withDescription("Thrown from handleResolvedAddresses(): " + e));
        }
      }
    }

    @Override
    public void onError(Status error) {
      checkArgument(!error.isOk(), "the error status must not be OK");
      // 通知 balancer 解析错误
      balancer.handleNameResolutionError(error);
    }
}
```

### name resolver 的 start()的触发

而类NameResolverListenerImpl只有一个地方使用，在方法 exitIdleMode() 中：

```java
LoadBalancer<ClientTransport> exitIdleMode() {
    synchronized (lock) {
	  ......
      if (loadBalancer != null) {
        // 如果loadBalancer不为空就直接return
        // 意味着如果想继续后面的代码，就必须loadBalancer为空才行
        return loadBalancer;
      }
      balancer = loadBalancerFactory.newLoadBalancer(nameResolver.getServiceAuthority(), tm);
      this.loadBalancer = balancer;
      resolver = this.nameResolver;
    }
    class NameResolverStartTask implements Runnable {
      @Override
      public void run() {
        // 这将在 LoadBalancer 和 NameResolver 中触发一些不平凡的工作
        // 不想在lock里面做(因此封装成task扔给scheduledExecutor)
        resolver.start(new NameResolverListenerImpl(balancer));
      }
    }

    scheduledExecutor.execute(new NameResolverStartTask());
    return balancer;
}
```

在 NameResolverStartTask 中才开始调用 name resolver 的 start() 方法启动 name resolver 的工作。而搜索中发现，这里是 name resolver 的 start() 方法的唯一一个使用的地方。也就是说，这是 name resolver 开始工作的唯一入口。

因此，name resolver 要开始工作，需要两个条件：

1. exitIdleMode() 方法被调用
2. loadBalancer 属性为null

exitIdleMode() 方法有两个被调用的地方：

1. inUseStateAggregator中，当发现 Channel 的状态转为使用中时：

    ```java
    final InUseStateAggregator<Object> inUseStateAggregator = new InUseStateAggregator<Object>() {
        ......
        void handleInUse() {
          exitIdleMode();
        }
    }
    ```

2. ClientTransportProvider的get()方法，通过exitIdleMode()方法获取balancer，然后通过balancer获取ClientTransport：

    ```java
    private final ClientTransportProvider transportProvider = new ClientTransportProvider() {
        @Override
        public ClientTransport get(CallOptions callOptions) {
          LoadBalancer<ClientTransport> balancer = exitIdleMode();
          if (balancer == null) {
            return SHUTDOWN_TRANSPORT;
          }
          return balancer.pickTransport(callOptions.getAffinity());
        }
    };
    ```

	这里返回的 ClientTransport 在RealChannel的ClientCall()方法中使用,被传递给新创建的ClientCallImpl：

    ```java
    private class RealChannel extends Channel {

    public <ReqT, RespT> ClientCall<ReqT, RespT> newCall(MethodDescriptor<ReqT, RespT> method, CallOptions callOptions) {
      ......
      return new ClientCallImpl<ReqT, RespT>(......
          transportProvider,......)
    }
	```

	而ClientCallImpl中这个get()方法只有一个地方被调用，在ClientCallImpl的start()方法中：

    ```java
    public void start(final Listener<RespT> observer, Metadata headers) {
    	......
        ClientTransport transport = clientTransportProvider.get(callOptions);
    }
    ```

### name resolver 的总结

name resolver 的 start() 方法，也就是 name resolver 要开始解析name的这个工作，只有两种情况下开始：

1. 第一次RPC请求： 此时要调用Channel的 newCall() 方法得到ClientCall的实例，然后调 ClientCall 的 start()方法，期间获取ClientTransport时激发一次 name resolver 的 start()
2. 如果开启了空闲模式：则在每次 Channel 从空闲模式退出，进入使用状态时，再激发一次 name resolver 的 start()

