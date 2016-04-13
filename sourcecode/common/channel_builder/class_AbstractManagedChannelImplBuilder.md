类AbstractManagedChannelImplBuilder
===================================

类AbstractManagedChannelImplBuilder 是 channel builder的基类。

# 类定义

```java
package io.grpc.internal;
public abstract class AbstractManagedChannelImplBuilder
        <T extends AbstractManagedChannelImplBuilder<T>> extends ManagedChannelBuilder<T> {}
```

> 注： package从 ManagedChannelBuilder 的 `io.grpc` 变成 `io.grpc.internal` 了。

# 类属性和方法实现

## executor

```java
@Nullable
private Executor executor;

@Override
public final T directExecutor() {
	return executor(MoreExecutors.directExecutor());
}

@Override
public final T executor(Executor executor) {
    this.executor = executor;
    return thisT();
}
```

executor(Executor executor)方法只是一个简单赋值，而directExecutor()则调用 MoreExecutors.directExecutor() 方法得到 direct executor。

> executor 在 build() 方法中被传递给新创建的 ManagedChannelImpl() 实例。

## 访问地址相关

和访问地址相关的三个属性：

1. target
2. directServerAddress
3. nameResolverFactory

```java
private final String target;
@Nullable
private final SocketAddress directServerAddress;
@Nullable
private NameResolver.Factory nameResolverFactory;
```

使用target来构造实例：

```java
protected AbstractManagedChannelImplBuilder(String target) {
    this.target = Preconditions.checkNotNull(target);
    // directServerAddress 设置为null
    this.directServerAddress = null;
}
```

使用directServerAddress来构造实例：

```java
private static final String DIRECT_ADDRESS_SCHEME = "directaddress";
protected AbstractManagedChannelImplBuilder(SocketAddress directServerAddress, String authority) {
	// target被设置为 directaddress：///directServerAddress
    this.target = DIRECT_ADDRESS_SCHEME + ":///" + directServerAddress;
    this.directServerAddress = directServerAddress;
    // nameResolverFactory 使用 DirectAddressNameResolverFactory
    this.nameResolverFactory = new DirectAddressNameResolverFactory(directServerAddress, authority);
}
```

设置nameResolverFactory的函数，注意NameResolverFactory和directServerAddress是互斥的：

```java
@Override
public final T nameResolverFactory(NameResolver.Factory resolverFactory) {
	// 如果设置directServerAddress，就不能再使用NameResolverFactory
    Preconditions.checkState(directServerAddress == null,
        "directServerAddress is set (%s), which forbids the use of NameResolverFactory",
        directServerAddress);
    this.nameResolverFactory = resolverFactory;
    return thisT();
}
```

> target 和 nameResolverFactory 在 build() 方法中被传递给新创建的 ManagedChannelImpl() 实例。

## user agent

```java
@Override
@Nullable
private String userAgent;

public final T userAgent(String userAgent) {
    this.userAgent = userAgent;
    return thisT();
}
```

> user agent 只是做了一个简单赋值，然后在 build() 方法中被传递给新创建的 ManagedChannelImpl() 实例。

## authorityOverride

```java
@Nullable
private String authorityOverride;

@Override
public final T overrideAuthority(String authority) {
    this.authorityOverride = checkAuthority(authority);
    return thisT();
}
```

简单做了个赋值，然后在build()方法中，在构建AuthorityOverridingTransportFactory时使用：

```java
@Override
public ManagedChannelImpl build() {
ClientTransportFactory transportFactory = new AuthorityOverridingTransportFactory(
    buildTransportFactory(), authorityOverride);
    ......
}
```


```java
@Override
public ManagedChannelImpl build() {
ClientTransportFactory transportFactory = new AuthorityOverridingTransportFactory(
    buildTransportFactory(), authorityOverride);
    ......
}
```

```java
AuthorityOverridingTransportFactory是一个内部类，以装饰模式包裹了一个ClientTransportFactory的实例，将请求都代理给这个包装的ClientTransportFactory实例，

private static class AuthorityOverridingTransportFactory implements ClientTransportFactory {
    final ClientTransportFactory factory;
    @Nullable final String authorityOverride;

    AuthorityOverridingTransportFactory(
        ClientTransportFactory factory, @Nullable String authorityOverride) {
      this.factory = factory;
      // 传递authorityOverride进来
      this.authorityOverride = authorityOverride;
    }

    @Override
    public ManagedClientTransport newClientTransport(SocketAddress serverAddress,
        String authority) {
        // 在这里做覆盖，如果authorityOverride设置了，就覆盖原有的authority
      return factory.newClientTransport(
          serverAddress, authorityOverride != null ? authorityOverride : authority);
    }
    ......
```

> 注： 感觉应该在构建AuthorityOverridingTransportFactory之前做authorityOverride的null检查，如果是null就没有必要构建AuthorityOverridingTransportFactory，直接用buildTransportFactory()返回的就可以了。
>
> 提交了一个pull request 给grpc项目，看看他们的意见如何： https://github.com/grpc/grpc-java/pull/1666


