类CallOptions
============

类CallOptions是新的RPC调用的运行时选项的集合。

## 类定义

```java
package io.grpc;
@Immutable
public final class CallOptions {
}
```

注意@Immutable标签，这个CallOptions是不可变类。

## 属性和构造函数

CallOptions的代码注释中有讲到：虽然CallOptions是不可变类，但是它的属性并没有声明为final。这样可以在构造函数之外赋值，否则就需要在构造函数中给出长长的一个参数列表。

属性有下面5个，其中有几个声明为容许为null：

```java
  private Long deadlineNanoTime;
  private Executor executor;

  @Nullable
  private String authority;

  @Nullable
  private RequestKey requestKey;

  @Nullable
  private String compressorName;
```

构造函数比较有意思，第二个版本的构造函数实现了复制功能，相当于创建了一个和给定实例数据完全相同的另一个对象实例：

```java
  private CallOptions() {
  }

  private CallOptions(CallOptions other) {
    deadlineNanoTime = other.deadlineNanoTime;
    authority = other.authority;
    requestKey = other.requestKey;
    executor = other.executor;
    compressorName = other.compressorName;
  }
```

为了不使用传统的不可变类的实现方式(属性声明为final，构造函数中一一赋值)，CallOptions中每次赋值都要使用上面的复制构造函数创建一个新的实例对象，然后返回新的实例对象(旧有实例就相当于抛弃了？):

```java
public CallOptions withDeadlineNanoTime(@Nullable Long deadlineNanoTime) {
    CallOptions newOptions = new CallOptions(this);
    newOptions.deadlineNanoTime = deadlineNanoTime;
    return newOptions;
}
```

有个疑惑，按照上面的实现机制，如果CallOptions的使用者要创建一个有多次复制的CallOptions示例，就必须如下编码：

```java
CallOptions callOptions = CallOptions.DEFAULT
	.withDeadlineNanoTime(***)
    .withAuthority(***)
    .withCompression(***)
    .withRequestKey(***)
```

这样实际会创建多次(取决于with()方法的调用次数)实例。不过从grpc的代码实现上看，每次只是在新建一个stub实例时才传入一个 CallOptions 实例，之后的每次调用都将重用这个 CallOptions 实例，所以实际上只是开始时创建多次实例，之后运行时就不再创建。

## 属性的详细含义

### deadline / 最后期限

withDeadlineNanoTime() 返回新的一个CallOptions，最后期限为在给定的绝对值。这个绝对值以纳秒为单位，和每次 System.nanoTime() 的时间一致：

```java
public CallOptions withDeadlineNanoTime(@Nullable Long deadlineNanoTime) {
    CallOptions newOptions = new CallOptions(this);
    newOptions.deadlineNanoTime = deadlineNanoTime;
    return newOptions;
}
```

withDeadlineAfter()方法返回一个最后期限为在给定期限之后的新的CallOptions， 可以看到实现的方式就是取当前的 System.nanoTime() 然后添加给定的期限：

```java
public CallOptions withDeadlineAfter(long duration, TimeUnit unit) {
	return withDeadlineNanoTime(System.nanoTime() + unit.toNanos(duration));
}
```

### executor

设置新的executor，用来覆盖 ManagedChannelBuilder.executor 指定的默认 executor 。

### requestKey

用于基于亲和度(affinity-based)的路由的request key。

### authority / 权限

覆盖 channel声明要连接到的 HTTP/2 的authority。**这不是普遍安全的**。覆盖容许高级用户为多个服务重用一个单一的channel，甚至这些服务在不同的域名上托管。这将假设服务器是虚拟托管了多个域名并被授权继续这样做。服务提供者极少作出这样的授权。此时，覆盖值没有安全认证，例如保证authority匹配服务器的TLS证书。

### compressorName / 压缩器名称

设置为调用使用的压缩方式。压缩器必须是在 CompressorRegistry 已知的有效名称。








