类NettyChannelBuilder
====================

在构建使用netty transport的channel的过程中，用来帮助简化d的builder。

# 类定义

```java
@ExperimentalApi("https://github.com/grpc/grpc-java/issues/1784")
public class NettyChannelBuilder extends AbstractManagedChannelImplBuilder<NettyChannelBuilder> {}
```

> 注: 都1.0.0-pre2了居然还是 @ExperimentalApi......


## 方法实现

### 最重要的buildTransportFactory()

```java
  @Override
protected ClientTransportFactory buildTransportFactory() {
	return new NettyTransportFactory(channelType, negotiationType, protocolNegotiator, sslContext, eventLoopGroup, flowControlWindow, maxMessageSize, maxHeaderListSize);
}
```

new了一个NettyTransportFactory实例，然后给了一堆参数。

下面看看各个参数的含义和传递/使用方式。

### channelType

使用的channel type，默认使用NioSocketChannel。貌似一般也都是用这个了。

```java
private Class<? extends Channel> channelType = NioSocketChannel.class;

public final NettyChannelBuilder channelType(Class<? extends Channel> channelType) {
    this.channelType = Preconditions.checkNotNull(channelType);
    return this;
}
```

### negotiationType

协商类型，具体看 `io.grpc.netty.NegotiationType` 这个枚举的说明。

NegotiationType 用于定义启动HTTP/2的协商方式：

- TLS： 使用 `TLS ALPN/NPN` 协商，假定是 SSL 连接
- PLAINTEXT_UPGRADE： 使用 HTTP 升级协议为 plaintext(非SSL)从 HTTP/1.1 升级到 HTTP/2
- PLAINTEXT: 假设连接是 plaintext(非SSL) 而且远程端点直接支持HTTP2.而不需要升级

从实现代码上看，默认是TLS，然后可以通过方法设置：

```java
private NegotiationType negotiationType = NegotiationType.TLS;

public final NettyChannelBuilder negotiationType(NegotiationType type) {
negotiationType = type;
return this;
}
```

此外还有一个usePlaintext()方法，指明使用 Plaintext， 然后通过参数 skipNegotiation 来指定是否需要跳过协商过程。这个方法等同于调用negotiationType(PLAINTEXT)或negotiationType(PLAINTEXT_UPGRADE),取决于参数取值：

```java
@Override
public final NettyChannelBuilder usePlaintext(boolean skipNegotiation) {
    if (skipNegotiation) {
      negotiationType(NegotiationType.PLAINTEXT);
    } else {
      negotiationType(NegotiationType.PLAINTEXT_UPGRADE);
    }
    return this;
}
```

从代码上看，只是一个简单的if判断，这个方法可以认为是 negotiationType()方法的一个可读性稍好的版本，存在的价值只是为了方便和易读。

### protocolNegotiator

设置使用的protocolNegotiator，如果不是null，则覆盖negotiationType(NegotiationType) 或者 usePlaintext(boolean):

```java
private ProtocolNegotiator protocolNegotiator;

@Internal
public final NettyChannelBuilder protocolNegotiator(
      @Nullable ProtocolNegotiator protocolNegotiator) {
    this.protocolNegotiator = protocolNegotiator;
    return this;
}
```

所谓覆盖，是这样实现的，在newClientTransport()方法中，先判断 protocolNegotiator：

```java
public ManagedClientTransport newClientTransport(
    SocketAddress serverAddress, String authority) {
    ......
    ProtocolNegotiator negotiator = protocolNegotiator != null ? protocolNegotiator :
      createProtocolNegotiator(authority, negotiationType, sslContext);
    return newClientTransport(serverAddress, authority, negotiator);
}
```

只有 protocolNegotiator 为null时，才会通过使用 authority / negotiationType / sslContext 参数来创建ProtocolNegotiator。如果不为null，则直接使用，无视上述几个参数。

### sslContext

可以设置 SL/TLS 上下文来使用，替代系统默认。必须已经用 GrpcSslContexts 配置过，但是选项可以被覆盖。

这个参数只是简单赋值，在上面的newClientTransport()方法中使用，注意同样的，如果protocolNegotiator被设置(不为null)，则这个sslContext将被忽略。

### eventLoopGroup

可以提供一个 EventGroupLoop 给netty传输使用。

这是一个可选参数。在channel构建时如果用户没有提供 EventGroupLoop ，builder将使用默认，而这个默认是静态的。注意 channel 不会为给定的 EventGroupLoop 负责，在需要时调用者有责任关闭它。

```java
@Nullable
private EventLoopGroup eventLoopGroup;

public final NettyChannelBuilder eventLoopGroup(@Nullable EventLoopGroup eventLoopGroup) {
    this.eventLoopGroup = eventLoopGroup;
    return this;
}
```

所谓使用静态的默认，代码实现是这样的：

```java
usingSharedGroup = group == null;
if (usingSharedGroup) {
	// The group was unspecified, using the shared group.
	this.group = SharedResourceHolder.get(Utils.DEFAULT_WORKER_EVENT_LOOP_GROUP);
} else {
	this.group = group;
}
```

如果没有设置，则取 SharedResourceHolder.get(Utils.DEFAULT_WORKER_EVENT_LOOP_GROUP)， 而DEFAULT_WORKER_EVENT_LOOP_GROUP 是这样定义的：

```java
class Utils {
  public static final Resource<EventLoopGroup> DEFAULT_BOSS_EVENT_LOOP_GROUP =
      new DefaultEventLoopGroupResource(1, "grpc-default-boss-ELG");
  public static final Resource<EventLoopGroup> DEFAULT_WORKER_EVENT_LOOP_GROUP =
      new DefaultEventLoopGroupResource(0, "grpc-default-worker-ELG");
```

构造函数中 worker的 numEventLoops 参数设置为0, 而另一个 boss 的 numEventLoops 参数设置为1。这会影响两个EventLoopGroup(可以理解为netty的线程池)的线程数，具体代码如下：

```java
@Override
public EventLoopGroup create() {
	......
    int parallelism = numEventLoops == 0
      ? Runtime.getRuntime().availableProcessors() * 2 : numEventLoops;
    return new NioEventLoopGroup(parallelism, threadFactory);
}
```

可以看到默认的 DEFAULT_WORKER_EVENT_LOOP_GROUP 的线程数为当前系统cpu数量的两倍。

> 注： 这个 SharedResourceHolder 的实现挺有意思，可以拿来在需要时参考一下，甚至直接搬过来用。

### flowControlWindow / maxMessageSize / maxHeaderListSize

这三个参数只是简单赋值和传递，然后给出了缺省的默认值：

```java
private int flowControlWindow = DEFAULT_FLOW_CONTROL_WINDOW;
private int maxMessageSize = DEFAULT_MAX_MESSAGE_SIZE;
private int maxHeaderListSize = GrpcUtil.DEFAULT_MAX_HEADER_LIST_SIZE;
```












