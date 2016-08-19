# 类 DnsNameResolver

## 类定义

DnsNameResolver 是 `io.grpc.internal` 下的类，包级私有，通过DnsNameResolverFactory类来创建。

研究一下它的实现，以便理解 NameResolver 的使用。

```java
package io.grpc.internal;

class DnsNameResolver extends NameResolver {}
```

## 属性和构造函数

属性比较多，先只看和 URI 相关的几个属性 authority/host/port：

```java
private final String authority;
private final String host;
private final int port;

DnsNameResolver(@Nullable String nsAuthority, String name, Attributes params,
      Resource<ScheduledExecutorService> timerServiceResource,
      Resource<ExecutorService> executorResource) {

    // 必须在 name 前加上"//"，否则将被当成含糊的URI，导致生成的URI的authority和host会变成null。
    URI nameUri = URI.create("//" + name);
    // 从生成的URI中得到 authority 并赋值，如果为空则报错
    authority = Preconditions.checkNotNull(nameUri.getAuthority(),
        "nameUri (%s) doesn't have an authority", nameUri);
    // 同样方式得到host，如果为空则报错
    host = Preconditions.checkNotNull(nameUri.getHost(), "host");
    // 端口特殊一点，因为可以通过params参数传入默认端口
    if (nameUri.getPort() == -1) {
      // 如果name中没有包含端口，则尝试从参数中获取默认端口
      Integer defaultPort = params.get(NameResolver.Factory.PARAMS_DEFAULT_PORT);
      if (defaultPort != null) {
        port = defaultPort;
      } else {
        throw new IllegalArgumentException(
            "name '" + name + "' doesn't contain a port, and default port is not set in params");
      }
    } else {
      port = nameUri.getPort();
    }
  }
```

构造函数中还传入了两个和Executor相关的参数，构造函数只是简单的保存起来：

```java
private final Resource<ScheduledExecutorService> timerServiceResource;
private final Resource<ExecutorService> executorResource;

DnsNameResolver(@Nullable String nsAuthority, String name, Attributes params,
	......
    this.timerServiceResource = timerServiceResource;
    this.executorResource = executorResource;
    ......
}
```

## 方法实现

### NameResolver 定义的方法

#### getServiceAuthority()

按照要求，这个方法直接返回在构造函数中就设置好的 authority 属性。考虑到 authority 属性是 final 的，因此也满足 NameResolver 接口中要求的：实现必须本地生成它，而且必须保持不变。使用同样的参数从同一个的 factory 中创建出来的 NameResolver 必须返回相同的 authority 。

```java
public final String getServiceAuthority() {
	return authority;
}
```

#### start()

按照要求，start() 开始解析。listener 用于接收目标的更新。

实现中，this.listener 属性用于保存传入的 listener ，而在 start() 时，timerService 和 executor 两个属性才会从之前传入的 timerServiceResource/executorResource 这两个SharedResourceHolder中获取实际的对象实例。

```java
public final synchronized void start(Listener listener) {
	// 先检查之前没有start过，判断依据是 this.listener 是否复制
    Preconditions.checkState(this.listener == null, "already started");
    timerService = SharedResourceHolder.get(timerServiceResource);
    executor = SharedResourceHolder.get(executorResource);
    // 为 this.listener 赋值，配合上面的检查
    this.listener = Preconditions.checkNotNull(listener, "listener");
    resolve();
}
```

resolve()方法开始做实际的解析，具体内容后面再看。

#### refresh()

DnsNameResolver 的 refresh() 实现直接调用了 resolve() 方法，和 start() 方法相比， start()中只是多了开始时的检查和resource获取，后面都是同样的调用resolve() 方法。

```java
public final synchronized void refresh() {
	// refresh()方法必须在 start() 之后调用，因此这里做了检查，同样判断依据是 listener 属性
    Preconditions.checkState(listener != null, "not started");
    resolve();
}
```

#### shutdown()

按照要求，shutdown()方法将停止解析，同时更新 listener 将会停止。实现中，将

shutdown属性用于标记是否已经关闭。

```java
@GuardedBy("this")
private boolean shutdown;

public final synchronized void shutdown() {
	// 通过 shutdown 属性来判断是否已经关闭
    if (shutdown) {
      return;
    }
    shutdown = true;
    if (resolutionTask != null) {
      // 如果 resolutionTask 不为空，取消它
      resolutionTask.cancel(false);
    }
    if (timerService != null) {
      // 释放 timerService 资源，返回null将清理属性 timerService
      timerService = SharedResourceHolder.release(timerServiceResource, timerService);
    }
    if (executor != null) {
      // 释放 executor 资源，返回null将清理属性 executor
      executor = SharedResourceHolder.release(executorResource, executor);
    }
}
```

## 解析的实际实现

从 resolve() 方法开始：

```java
private void resolve() {
	// resolving 属性和 shutdown 协助判断一下状态
    if (resolving || shutdown) {
      return;
    }
    // 将 resolutionRunnable 作为任务扔给executor
    // 然后方法返回，异步做解析
    // 这样 start()和 refresh() 方法就都是快速返回，异步解析后通过 listener 做数据更新
    executor.execute(resolutionRunnable);
}
```

继续看 resolutionRunnable 的实现，代码有点长，先排除细节代码，看主流程：

```java
private final Runnable resolutionRunnable = new Runnable() {
  @Override
  public void run() {
    ......

      try {
        // 1. 将host解析为多个IP地址，InetAddress格式
        inetAddrs = getAllByName(host);
      } catch (UnknownHostException e) {
		......
      }
      List<ResolvedServerInfo> servers =
          new ArrayList<ResolvedServerInfo>(inetAddrs.length);
      for (int i = 0; i < inetAddrs.length;i++){
        InetAddress inetAddr = inetAddrs[i];
        // 2. 将每个 InetAddress 格式的IP地址包装为 ResolvedServerInfo 对象
        servers.add(
            new ResolvedServerInfo(new InetSocketAddress(inetAddr, port), Attributes.EMPTY));
      }
      // 3. 通知listner，有数据更新
      savedListener.onUpdate(
          Collections.singletonList(servers), Attributes.EMPTY);

  }
};
```

主流程就是上面注释中的3个步骤：解析地址，包装格式，通知listener。注意这个工作是在异步线程中进行的，无法直接return结果，只能通过listener。

再继续看错误流程，如果解析地址失败：

```java
private final Runnable resolutionRunnable = new Runnable() {
  @Override
  public void run() {
    ......
    try {
    	inetAddrs = getAllByName(host);
    } catch (UnknownHostException e) {
    	// 遇到无法解析的情况
        synchronized (DnsNameResolver.this) {
            if (shutdown) {
                return;
            }
            // 因为在产品中 timerService 是一个单线程的 GrpcUtil.TIMER_SERVICE
            // 我们需要将这个阻塞的工作交给 executor
            resolutionTask = timerService.schedule(new LogExceptionRunnable(resolutionRunnableOnExecutor), 1, TimeUnit.MINUTES);
            }
            // 通知listener遇到错误
            savedListener.onError(Status.UNAVAILABLE.withCause(e));
            return;
        }
    }
    ......
};
```

出错了通知listener这个容易理解，前面的 resolutionTask 是什么呢？

```java
// 这是定义，一个标准的ScheduledFuture
private ScheduledFuture<?> resolutionTask;
......
// 唯一一个赋值的地方，就是当解析出错时
resolutionTask = timerService.schedule(new LogExceptionRunnable(resolutionRunnableOnExecutor), 1, TimeUnit.MINUTES);
```

通过扔给 timerService 一个一次性的 LogExceptionRunnable 的任务，要求延迟1分钟执行，然后得到这个 resolutionTask 的 feature 。这个 feature 在 shutdown()方法和 resolutionRunnable 的 run()方法中有细节处理。

继续看 LogExceptionRunnable 里面做了什么：

```java
// 对 Runnable 的简单包裹，用于记录它抛出的任何异常，在重新抛出之前
public final class LogExceptionRunnable implements Runnable {
  // 构造函数只是简单的保存传入的task
  public LogExceptionRunnable(Runnable task) {
    this.task = checkNotNull(task);
  }
  public void run() {
    try {
      // 执行task
      task.run();
    } catch (Throwable t) {
      // 捕获异常，先打印日志，这个是主要目的了
      log.log(Level.SEVERE, "Exception while executing runnable " + task, t);
      // 再重新抛出去
      throw t instanceof RuntimeException ? (RuntimeException) t : new RuntimeException(t);
    }
  }
}
```

只是简单的执行task并记录日志，所以关键还是看 resolutionRunnableOnExecutor 这里面的内容：

```java
private final Runnable resolutionRunnableOnExecutor = new Runnable() {
  @Override
  public void run() {
    synchronized (DnsNameResolver.this) {
      if (!shutdown) {
        // 再执行一次 resolutionRunnable，这里和 resolve() 函数差不多的功能
        executor.execute(resolutionRunnable);
      }
    }
  }
};
```

转了一圈又回来了：

1. resolutionRunnable 在解析失败时，就会给 timerService 安排一个一分钟之后执行的任务 resolutionTask
2. 在这个任务中，将重新用 executor 跑一次 resolutionRunnable
3. 如果继续解析失败，则循环上述过程，每分钟尝试一次

注意在 resolutionRunnable 中，每次发现 resolutionTask 存在就会先 cancel 掉它，然后置为null：

```java
if (resolutionTask != null) {
    resolutionTask.cancel(false);
    resolutionTask = null;
}
```

所以上述的循环，只是在每次解析失败时，一旦解析成功，就会跳出循环。
