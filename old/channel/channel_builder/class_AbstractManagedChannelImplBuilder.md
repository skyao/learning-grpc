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
    this.target = makeTargetStringForDirectAddress(directServerAddress);
    this.directServerAddress = directServerAddress;
    // nameResolverFactory 使用 DirectAddressNameResolverFactory
    this.nameResolverFactory = new DirectAddressNameResolverFactory(directServerAddress, authority);
}

@VisibleForTesting
static String makeTargetStringForDirectAddress(SocketAddress address) {
    try {
      // "directaddress:///address"
      return new URI(DIRECT_ADDRESS_SCHEME, "", "/" + address, null).toString();
    } catch (URISyntaxException e) {
      // It should not happen.
      throw new RuntimeException(e);
    }
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
	ClientTransportFactory transportFactory = buildTransportFactory();
    if (authorityOverride != null) {
      transportFactory = new AuthorityOverridingTransportFactory(
        transportFactory, authorityOverride);
    }
    ......
    return new ManagedChannelImpl(
        ......,transportFactory,......);
}
```

通过buildTransportFactory()方法得到ClientTransportFactory，然后传递给ManagedChannelImpl的构造函数。为了覆盖原来的Authority，实现了一个AuthorityOverridingTransportFactory内部类，以装饰模式包裹了一个ClientTransportFactory的实例，然后将请求都代理给这个包装的ClientTransportFactory实例：

```java
private static class AuthorityOverridingTransportFactory implements ClientTransportFactory {
    final ClientTransportFactory factory;
    final String authorityOverride;

    AuthorityOverridingTransportFactory(
        ClientTransportFactory factory, String authorityOverride) {
      this.factory = Preconditions.checkNotNull(factory, "factory should not be null");
      this.authorityOverride = Preconditions.checkNotNull(
        authorityOverride, "authorityOverride should not be null");
    }

	@Override
    public ConnectionClientTransport newClientTransport(SocketAddress serverAddress,
        String authority, @Nullable String userAgent) {
        // 在这里做覆盖，用authorityOverride覆盖原有的authority
      return factory.newClientTransport(serverAddress, authorityOverride, userAgent);
    }
    ......
```

在newClientTransport()方法中， 前面传递进来的 authorityOverride 派上用场了。

> 注： 之前AuthorityOverridingTransportFactory的实现有点小问题，我提交了一个pull request给grpc，后来被采纳，现在这里的代码已经合并到master，看着真亲切 :)： https://github.com/grpc/grpc-java/pull/1666

## nameResolverFactory

```java
@Nullable
private NameResolver.Factory nameResolverFactory;

protected AbstractManagedChannelImplBuilder(SocketAddress directServerAddress, String authority) {
    ......
    this.nameResolverFactory = new DirectAddressNameResolverFactory(directServerAddress, authority);
}

@Override
public final T nameResolverFactory(NameResolver.Factory resolverFactory) {
    Preconditions.checkState(directServerAddress == null,
        "directServerAddress is set (%s), which forbids the use of NameResolverFactory",
        directServerAddress);
    this.nameResolverFactory = resolverFactory;
    return thisT();
}
```

如前所述，如果设置了directServerAddress，则nameResolverFactory自动设置为 DirectAddressNameResolverFactory 。而且不能再设置 nameResolverFactory 。

nameResolverFactory在build()方法中使用，有一个null的检查，如果 nameResolverFactory 有设置则使用 nameResolverFactory， 否则使用默认的 NameResolverProvider.asFactory()：

```java
@Override
public ManagedChannelImpl build() {
	......
    NameResolver.Factory nameResolverFactory = this.nameResolverFactory;
    if (nameResolverFactory == null) {
      nameResolverFactory = NameResolverProvider.asFactory();
    }
    return new ManagedChannelImpl(
        ......,
        nameResolverFactory，
        getNameResolverParams()，
        ......);
}
```

> 注：这里的NameResolverProvider.asFactory()看不懂......

额外的getNameResolverParams()用来给子类使用(以override的方式)，可以传递更多参数给 NameResolver.Factory.newNameResolver() 方法。默认实现只是返回一个Attributes.EMPTY

```java
protected Attributes getNameResolverParams() {
	return Attributes.EMPTY;
}
```

## loadBalancerFactory

```java
@Nullable
private LoadBalancer.Factory loadBalancerFactory;

@Override
public final T loadBalancerFactory(LoadBalancer.Factory loadBalancerFactory) {
    Preconditions.checkState(directServerAddress == null,
        "directServerAddress is set (%s), which forbids the use of LoadBalancerFactory",
        directServerAddress);
    this.loadBalancerFactory = loadBalancerFactory;
    return thisT();
}

public ManagedChannelImpl build() {
    return new ManagedChannelImpl(
        ......,
        firstNonNull(loadBalancerFactory, DummyLoadBalancerFactory.getInstance()),
        ......);
}
```

和 nameResolverFactory 的使用非常类似，同样是 directServerAddress 设置后不能再用，使用的方式也是在build()中检查是否有设置，如果没有设置则默认使用 DummyLoadBalancerFactory 。

## decompressorRegistry 和 compressorRegistry

```java
public ManagedChannelImpl build() {
    return new ManagedChannelImpl(
        ......,
        firstNonNull(decompressorRegistry, DecompressorRegistry.getDefaultInstance()),
        firstNonNull(compressorRegistry, CompressorRegistry.getDefaultInstance()),
        ......);
}
```

decompressorRegistry 和 compressorRegistry 基本就是个简单设置+ 在build()方法中传递给ManagedChannelImpl()，如果没有设置则用默认值。

# 关键方法

## 抽象方法 buildTransportFactory()

定义抽象方法 buildTransportFactory()， 用于子类实现这个方法来为这个channel提供 ClientTransportFactory 。这个方法只对Transport 的实现者有意义，而不应该被普通用户使用。

```java
protected abstract ClientTransportFactory buildTransportFactory();
```

返回的 ClientTransportFactory 将在 build() 方法中传递给 ManagedChannelImpl() 的构造函数。











