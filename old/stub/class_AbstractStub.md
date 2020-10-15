类AbstractStub
=============

类AbstractStub是stub实现的通用基类。

类AbstractStub也是生成代码中的stub类的通用基类。这个类容许重定义，例如，添加拦截器到stub。

## 类定义

```java
package io.grpc.stub;
public abstract class AbstractStub<S extends AbstractStub<S>> {
}
```

## 属性和构造函数

类AbstractStub有两个属性：

1. Channel channel
2. CallOptions callOptions

```java
private final Channel channel;
private final CallOptions callOptions;

protected AbstractStub(Channel channel) {
	this(channel, CallOptions.DEFAULT);
}

protected AbstractStub(Channel channel, CallOptions callOptions) {
    this.channel = channel;
    this.callOptions = callOptions;
}
```

> 注： 类CallOptions 的内容见 [这里](../channel/class_CallOptions.md)

## 方法

### build()抽象方法

定义了抽象方法build()方法来返回一个新的stub，使用给定的Channel和提供的方法配置。

```java
protected abstract S build(Channel channel, CallOptions callOptions);
```

- channel： 返回的新的stub将使用这个Channel来做通讯。
- callOptions： 运行时调用选项，将被应用于这个stub的每次调用。(也就是说这个不可变的callOptions实例之后将在每次调用中重用)

### with方法族

定义有多个with×××()方法，通过创建新的 CallOptions 实例，然后调用上面的build()方法来返回一个新的stub：

```java
public final S withDeadlineNanoTime(@Nullable Long deadlineNanoTime) {
	return build(channel, callOptions.withDeadlineNanoTime(deadlineNanoTime));
}
public final S withDeadlineAfter(long duration, TimeUnit unit) {
	return build(channel, callOptions.withDeadlineAfter(duration, unit));
}
@ExperimentalApi
public final S withCompression(String compressorName) {
	return build(channel, callOptions.withCompression(compressorName));
}
```

也可以替换使用新的Channel，或者在原有的Channel上添加拦截器：

```java
public final S withChannel(Channel newChannel) {
	return build(newChannel, callOptions);
}
public final S withInterceptors(ClientInterceptor... interceptors) {
	return build(ClientInterceptors.intercept(channel, interceptors), callOptions);
}
```

