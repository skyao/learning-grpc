# 空闲模式

在最新的版本(1.0.0-pre1)中，gRPC 引入了 Channel 的空闲模式(idle mode)。

## 工作方式(初步)

在 InUseStateAggregator 中，控制 Channel (准备)进入空闲模式和退出空闲模式：

```java
final InUseStateAggregator<Object> inUseStateAggregator = new InUseStateAggregator<Object>() {

    @Override
    void handleInUse() {
      // 被使用时，就退出空闲模式
      exitIdleMode();
    }

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

/** 进入空闲模式的超时时间 **/
private final long idleTimeoutMillis;
```

### idle timer

```java
// 不能为null，一定会被 channelExecutor 使用
@Nullable
private ScheduledFuture< ?> idleModeTimerFuture;

// 不能为null，一定会被 channelExecutor 使用
@Nullable
private IdleModeTimer idleModeTimer;
```

类 IdleModeTimer 实际是一个 Runnable (按说取名应该是IdleModeTimerTask才对)，用于使 Channel 进入空闲模式，并关闭以下内容：

1. Name Resolver
2. Load Balancer
3. subchannelPicker

```java
// 由 channelExecutor 运行
private class IdleModeTimer implements Runnable {
    // 仅仅由 channelExecutor 修改
    boolean cancelled;

    @Override
    public void run() {
        if (cancelled) {
        	// 检测到竞争： 这个任务在 channelExecutor 中安排在 cancelIdleTimer() 之前
            // 需要取消timer
        	return;
        }
        log.log(Level.FINE, "[{0}] Entering idle mode", getLogId());
        // nameResolver 和 loadBalancer 保证不为null。如果他们中的任何一个是 null ，
        // 要不是 idleModeTimer 在没有退出空闲模式的情况下运行了两次，
        // 就是 shutdown() 中的任务没有取消 idleModeTimer，两者都是bug
        // 1. 关闭当前的 nameResolver, 并重新创建一个新的 nameResolver
        nameResolver.shutdown();
        nameResolver = getNameResolver(target, nameResolverFactory, nameResolverParams);
        // 2. 关闭当前的 loadBalancer 并置为 null
        loadBalancer.shutdown();
        loadBalancer = null;
        // 3. 当前 subchannelPicker 置为 null
        subchannelPicker = null;
    }
}
```

> 注：没有看懂，为什么要调用 getNameResolver()方法来创建新的 nameResolver？

## 退出空闲模式

让 Channel 退出空闲模式，如果它处于空闲模式中。返回一个新的可以用于处理新请求的 LoadBalancer。如果 Channel 被关闭则返回null。

```java
//让 channel 退出空闲模式，如果它处于空闲模式
//必须由 channelExecutor 调用
void exitIdleMode() {
    if (shutdown.get()) {
    	return;
    }
    if (inUseStateAggregator.isInUse()) {
        // 立即取消 timer，这样由于 timer 导致的竞争不会将 channel 设置为空闲
        // 注： 如果正在使用中，则后台会有一个idle timer，需要取消这个timer
        cancelIdleTimer();
    } else {
        // exitIdleMode() 可能在 inUseStateAggregator.handleNotInUse() 之外被调用，此时isInUse() == false
        // 在这种情况下我们依然需要安排 timer
        rescheduleIdleTimer();
    }
    if (loadBalancer != null) {
    	return;
    }
    log.log(Level.FINE, "[{0}] Exiting idle mode", getLogId());
    // 1. 创建新的 loadBalancer
    LbHelperImpl helper = new LbHelperImpl(nameResolver);
    helper.lb = loadBalancerFactory.newLoadBalancer(helper);
    this.loadBalancer = helper.lb;

	// 2. 创建新的 NameResolverListener
    NameResolverListenerImpl listener = new NameResolverListenerImpl(helper);
    try {
    	nameResolver.start(listener);
    } catch (Throwable t) {
    	listener.onError(Status.fromThrowable(t));
    }
}
```

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
    idleModeTimerFuture = scheduledExecutor.schedule(
        new LogExceptionRunnable(new Runnable() {
            @Override
            public void run() {
              channelExecutor.executeLater(idleModeTimer).drain();
            }
          }),
        idleTimeoutMillis, TimeUnit.MILLISECONDS);
}
```


## 控制空闲模式的开启

上面可以看到空闲模式的开启由属性 idleTimeoutMillis 控制，具体看这个属性的使用：

```java
static final long IDLE_TIMEOUT_MILLIS_DISABLE = -1;

// final属性
private final long idleTimeoutMillis;

ManagedChannelImpl(String target, ......, long idleTimeoutMillis, ......) {
	......
    if (idleTimeoutMillis == IDLE_TIMEOUT_MILLIS_DISABLE) {
    	this.idleTimeoutMillis = idleTimeoutMillis;
    } else {
    	checkArgument(idleTimeoutMillis >= AbstractManagedChannelImplBuilder.IDLE_MODE_MIN_TIMEOUT_MILLIS, "invalid idleTimeoutMillis %s", idleTimeoutMillis);
    	this.idleTimeoutMillis = idleTimeoutMillis;
    }
    // 有效性检查，要开启就必须 > 0，要关闭就设置为 IDLE_TIMEOUT_MILLIS_DISABLE
	checkArgument(idleTimeoutMillis > 0 || idleTimeoutMillis == IDLE_TIMEOUT_MILLIS_DISABLE, "invalid idleTimeoutMillis %s", idleTimeoutMillis);
    // 构造函数中赋值
    this.idleTimeoutMillis = idleTimeoutMillis;
}
```

再往上追，控制权在构造 ManagedChannelImpl 时，只有一个调用的地方, AbstractManagedChannelImplBuilder 中的build()方法：

```java
public ManagedChannelImpl build() {
	......
    return new ManagedChannelImpl(......,idleTimeoutMillis,......);
}
```

属性 idleTimeoutMillis 在 build() 时传入，而这个属性的设置是通过 idleTimeout() 方法：

```java
// 默认值为 IDLE_TIMEOUT_MILLIS_DISABLE
// 因此如果不调用 idleTimeout() , 默认是不会开启空闲模式的
private long idleTimeoutMillis = IDLE_TIMEOUT_MILLIS_DISABLE;

// 最大的空闲超时时间，比这个还大则将禁用空闲模式
// 最大空闲30天，也够夸张的
static final long IDLE_MODE_MAX_TIMEOUT_DAYS = 30;

// 默认超时时间
static final long IDLE_MODE_DEFAULT_TIMEOUT_MILLIS = TimeUnit.MINUTES.toMillis(30);

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


