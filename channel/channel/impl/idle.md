# Channel的空闲模式

在最新的版本(1.0.0-pre1)中，gRPC 引入了 Channel 的空闲模式(idle mode)。

## 工作方式

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

