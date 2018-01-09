# 类ManagedChannel

类ManagedChannel 在 Channel 的基础上提供生命周期管理的功能。

实际实现式就是添加了 shutdown()/shutdownNow() 方法用于关闭 Channel，isShutdown()/isTerminated() 方法用于检测 Channel 状态， 以及 awaitTermination() 方法用于等待关闭操作完成。

## 类定义

```java
package io.grpc;

public abstract class ManagedChannel extends Channel {}
```

依然是 abstract class，还继续在 package io.grpc中。

## 类方法

### shutdown()方法

```java
public abstract ManagedChannel shutdown();
```

发起一个有组织的关闭，期间已经存在的调用将继续，而新的调用将被立即取消。

### shutdownNow()方法

```java
public abstract ManagedChannel shutdownNow();
```

发起一个强制的关闭，期间已经存在的调用和新的调用都将被取消。虽然是强制，关闭过程依然不是即可生效;如果在这个方法返回后立即调用 isTerminated() 方法，将可能返回 false 。

### isShutdown()方法

```java
public abstract boolean isShutdown();
```

返回channel是否是关闭。关闭的channel立即取消任何新的调用，但是将继续有一些正在处理的调用。

### isTerminated()方法

```java
public abstract boolean isTerminated();
```

返回channel是否是结束。结束的 channel 没有运行中的调用，并且相关的资源已经释放（像TCP连接）。

### awaitTermination()方法

```java
public abstract boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException;
```

等待 channel 变成结束，如果超时时间达到则放弃。

返回值表明 channel 是否结束，和 isTerminated() 方法一致。

