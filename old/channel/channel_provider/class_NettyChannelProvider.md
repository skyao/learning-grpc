类NettyChannelProvider和OkHttpChannelProvider
=====================

## 类NettyChannelProvider

用于创建 NettyChannelBuilder 实例的provider。

```java
@Internal
public final class NettyChannelProvider extends ManagedChannelProvider {
  @Override
  public boolean isAvailable() {
    // 总是返回true
    return true;
  }

  @Override
  public int priority() {
    // 使用默认优先级 5
    return 5;
  }

  @Override
  public NettyChannelBuilder builderForAddress(String name, int port) {
    // 对接NettyChannelBuilder
    return NettyChannelBuilder.forAddress(name, port);
  }

  @Override
  public NettyChannelBuilder builderForTarget(String target) {
    // 对接NettyChannelBuilder
    return NettyChannelBuilder.forTarget(target);
  }
}
```

## 类OkHttpChannelProvider

用于创建 OkHttpChannelBuilder 实例的provider。

```java
@Internal
public final class OkHttpChannelProvider extends ManagedChannelProvider {

  @Override
  public boolean isAvailable() {
    return true;
  }

  @Override
  public int priority() {
    // 会更加当前环境动态调整优先级
    // 如果是 IS_RESTRICTED_APPENGINE 或者 android 环境下，调整为 8, 此时优先级高于 netty(默认为5)
    // 其他情况下为 3, 此时优先级低于 netty(默认为5)
    return (GrpcUtil.IS_RESTRICTED_APPENGINE || isAndroid()) ? 8 : 3;
  }

  @Override
  public OkHttpChannelBuilder builderForAddress(String name, int port) {
    return OkHttpChannelBuilder.forAddress(name, port);
  }

  @Override
  public OkHttpChannelBuilder builderForTarget(String target) {
    return OkHttpChannelBuilder.forTarget(target);
  }
}
```

IS_RESTRICTED_APPENGINE的判断逻辑如下(和 google appengine 有关，细节不追究了)：

```java
  public static final boolean IS_RESTRICTED_APPENGINE =
     "Production".equals(System.getProperty("com.google.appengine.runtime.environment"))
          && "1.7".equals(System.getProperty("java.specification.version"));
```

## ServiceProvider 的实现

为了实现jdk的ServiceProvider，以便被load()函数装载，okjava 和 netty 在打包的时候都会按照JDK SPI的要求在他们的jar文件中加入SPI的内容，以netty为例：

![](images/spi.png)

上图是 grpc-java 中 netty 子项目的相关文件，在resources下的 `META-INF.services` 目录，存在一个名为 `io.grpc.ManagedChannelProvider` 文件，其内容为：

	io.grpc.netty.NettyChannelProvider

## 总结

通过这种标准的SPI的方式，grpc实现了将 channel 的提供者和使用者分离并解藕。

