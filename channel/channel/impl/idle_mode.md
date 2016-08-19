# 空闲模式

在最新的版本(1.0.0-pre1)中，gRPC 引入了 Channel 的空闲模式(idle mode)。

## 工作方式(初步)

在 InUseStateAggregator 中，控制 Channel (准备)进入空闲模式和退出空闲模式：

```java
final InUseStateAggregator<Object> inUseStateAggregator = new InUseStateAggregator<Object>() {
    @Override
    Object getLock() {
      return lock;
    }

    @Override
    @GuardedBy("lock")
    void handleInUse() {
      // 被使用时，就退出空闲模式
      exitIdleMode();
    }

    @GuardedBy("lock")
    @Override
    void handleNotInUse() {
      if (shutdown) {
        return;
      }
      // 如果不被使用，就开始安排定时器准备进入空闲模式
      rescheduleIdleTimer();
    }
};
```

### idle 相关属性

```java
static final long IDLE_TIMEOUT_MILLIS_DISABLE = -1;

static final ClientTransport IDLE_MODE_TRANSPORT = new FailingClientTransport(Status.INTERNAL.withDescription("Channel is in idle mode"));

private final long idleTimeoutMillis;

// 保存进入空闲模式时被 shutdown (但是还没有terminated) 的Transports
@GuardedBy("lock")
private final HashSet<TransportSet> decommissionedTransports = new HashSet<TransportSet>();
```

### idle timer

```java
@GuardedBy("lock")
@Nullable
private ScheduledFuture< ?> idleModeTimerFuture;

@GuardedBy("lock")
@Nullable
private IdleModeTimer idleModeTimer;
```

类IdleModeTimer 实际是一个 Runnable (按说取名应该是IdleModeTimerTask才对)，用于使 Channel 进入空闲模式，并关闭以下内容：

1. Name Resolver
2. Load Balancer
3. 当前正在使用的 Transport

```java
private class IdleModeTimer implements Runnable {
    @GuardedBy("lock")
    boolean cancelled;

    @Override
    public void run() {
      ArrayList<TransportSet> transportsCopy = new ArrayList<TransportSet>();
      LoadBalancer<ClientTransport> savedBalancer;
      NameResolver oldResolver;
      synchronized (lock) {
        if (cancelled) {
          // Race detected: this task started before cancelIdleTimer() could cancel it.
          return;
        }
        // 进入空闲模式
        savedBalancer = loadBalancer;	// 保存当前的loadBalancer
        loadBalancer = null;			// 将loadBalancer设置为null
        oldResolver = nameResolver;		// 保存当前的nameResolver，然后再重新创建一个新的nameResolver
        nameResolver = getNameResolver(target, nameResolverFactory, nameResolverParams);
        transportsCopy.addAll(transports.values());	// 保存当前的transports
        transports.clear();		// 清理当前的transports
        decommissionedTransports.addAll(transportsCopy); // 将要关闭的 Transports 保存起来
      }
      for (TransportSet ts : transportsCopy) {
      	// 关闭当前所有的transports
        ts.shutdown();
      }
      // 关闭 Load Balancer
      savedBalancer.shutdown();
      // 关闭 Name Resolver
      oldResolver.shutdown();
    }
}
```

> 注：没有看懂，为什么要调用 getNameResolver()方法来创建新的 nameResolver？

## 退出空闲模式

让 Channel 退出空闲模式，如果它处于空闲模式中。返回一个新的可以用于处理新请求的 LoadBalancer。如果 Channel 被关闭则返回null。

```java
LoadBalancer<ClientTransport> exitIdleMode() {
    final LoadBalancer<ClientTransport> balancer;
    final NameResolver resolver;
    synchronized (lock) {
      if (shutdown) {
        return null;
      }
      if (inUseStateAggregator.isInUse()) {
        // 如果正在使用中，则后台会有一个idle timer，需要取消这个timer
        cancelIdleTimer();
      } else {
        // exitIdleMode()可能在 inUseStateAggregator 之外被调用
        // 这样就可能依然处于"未被使用"的状态
        // 如果是这样，我们启动timer，它将被迅速取消，如果 aggregator 收到实际请用请求。
        rescheduleIdleTimer();
      }
      if (loadBalancer != null) {
        return loadBalancer;
      }
      balancer = loadBalancerFactory.newLoadBalancer(nameResolver.getServiceAuthority(), tm);
      this.loadBalancer = balancer;
      resolver = this.nameResolver;
    }
    class NameResolverStartTask implements Runnable {
      @Override
      public void run() {
        // This may trigger quite a few non-trivial work in LoadBalancer and NameResolver,
        // we don't want to do it in the lock.
        resolver.start(new NameResolverListenerImpl(balancer));
      }
    }

    scheduledExecutor.execute(new NameResolverStartTask());
    return balancer;
}
```

> 注： 第一次看到在方法内这样写一个类（上面的NameResolverStartTask），汗，理由是什么？为什么不是直接写一个内部匿名类 `scheduledExecutor.execute(new Runnable(){...})` ?

### cancelIdleTimer()的实现

```java
private void cancelIdleTimer() {
    if (idleModeTimerFuture != null) {
      // 取消feture
      idleModeTimerFuture.cancel(false);
      // 设置timer为取消
      idleModeTimer.cancelled = true;
      // 然后都设置为null
      idleModeTimerFuture = null;
      idleModeTimer = null;
    }
}
```

### rescheduleIdleTimer()的实现

```java
private void rescheduleIdleTimer() {
    if (idleTimeoutMillis == IDLE_TIMEOUT_MILLIS_DISABLE) {
      // 这个检查控制着 空闲模式 的开启和关闭，具体分析见后面
      return;
    }
    // 取消现有的可能的timer，主要这个方法执行后 idleModeTimer 会被设置为null
    cancelIdleTimer();
    // 重新再构造一个timer
    idleModeTimer = new IdleModeTimer();
    // 重新再安排这个timer的执行
    idleModeTimerFuture = scheduledExecutor.schedule(new LogExceptionRunnable(idleModeTimer), idleTimeoutMillis, TimeUnit.MILLISECONDS);
}
```


## 控制空闲模式的开启

上面可以看到空闲模式的开启由属性 idleTimeoutMillis 控制，具体看这个属性的使用：

```java
static final long IDLE_TIMEOUT_MILLIS_DISABLE = -1;

// final属性
private final long idleTimeoutMillis;

ManagedChannelImpl(String target, ......, long idleTimeoutMillis, ......) {
    // 有效性检查，要开启就必须 > 0，要关闭就设置为 IDLE_TIMEOUT_MILLIS_DISABLE
	checkArgument(idleTimeoutMillis > 0 || idleTimeoutMillis == IDLE_TIMEOUT_MILLIS_DISABLE, "invalid idleTimeoutMillis %s", idleTimeoutMillis);
    // 构造函数中赋值
    this.idleTimeoutMillis = idleTimeoutMillis;
}
```

再往上追，控制权在构造 ManagedChannelImpl 时，只有一个调用的地方, AbstractManagedChannelImplBuilder中的build()方法：

```java
public ManagedChannelImpl build() {
	......
    return new ManagedChannelImpl(......,idleTimeoutMillis,......);
}
```

属性 idleTimeoutMillis 在build() 时传入，而这个属性的设置是通过 idleTimeout() 方法：

```java
// 默认值为 IDLE_TIMEOUT_MILLIS_DISABLE
// 因此如果不调用 idleTimeout() , 默认是不会开启空闲模式的
private long idleTimeoutMillis = ManagedChannelImpl.IDLE_TIMEOUT_MILLIS_DISABLE;

// 最大的空闲超时时间，比这个还大则将禁用空闲模式
// 最大空闲30天，也够夸张的
static final long IDLE_MODE_MAX_TIMEOUT_DAYS = 30;

// 最小空闲时间为1秒，如果设置的比这个还短则会设置为1秒
static final long IDLE_MODE_MIN_TIMEOUT_MILLIS = TimeUnit.SECONDS.toMillis(1);

public final T idleTimeout(long value, TimeUnit unit) {
	// 检查必须 > 0，
	checkArgument(value > 0, "idle timeout is %s, but must be positive", value);
    // We convert to the largest unit to avoid overflow
    if (unit.toDays(value) >= IDLE_MODE_MAX_TIMEOUT_DAYS) {
      // 检查如果超过最大值，则禁用空闲模式
      this.idleTimeoutMillis = ManagedChannelImpl.IDLE_TIMEOUT_MILLIS_DISABLE;
    } else {
      // 转为毫秒单位，如果比最小容许值(1秒)还小则取最小容许值
      this.idleTimeoutMillis = Math.max(unit.toMillis(value), IDLE_MODE_MIN_TIMEOUT_MILLIS);
    }
    return thisT();
}
```

## 工作方式(深入)

前面发现空闲模式的进入和退出是由 InUseStateAggregator 来进行控制的，而 InUseStateAggregator 的工作方式请见下一节。


