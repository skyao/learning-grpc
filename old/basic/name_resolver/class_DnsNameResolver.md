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
      Resource<ExecutorService> executorResource,
      ProxyDetector proxyDetector) {

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

按照要求，这个方法直接返回在构造函数中就设置好的 authority 属性。考虑到 authority 属性是 final 的，因此也满足 NameResolver 接口中要求的： "实现必须非阻塞式的生成它，而且必须保持不变。使用同样的参数从同一个的 factory 中创建出来的 NameResolver 必须返回相同的 authority "。

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
	// 先检查之前没有start过，判断依据是 this.listener 是否已有赋值
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

继续看 resolutionRunnable 的实现，代码有点长，先排除各种细节处理，只看主流程：

```java
private final Runnable resolutionRunnable = new Runnable() {
    @Override
    public void run() {
    	//忽略此处的状态检查代码和grpc proxy处理代码
        ......
        ResolutionResults resolvedInetAddrs;
        try {
        	// step1. 将host解析为ResolutionResults
        	// 新版本引入了一个 delegateResolver，细节稍后再看
            // 开始对 host 做 dns 解析，得到解析结果
        	resolvedInetAddrs = delegateResolver.resolve(host);
        } catch (Exception e) {
        	...... // 异常处理的流程后面再细看
            return;
        }
        // 解析出来的每个地址组成一个 EAG
        ArrayList<EquivalentAddressGroup> servers = new ArrayList<EquivalentAddressGroup>();
        for (InetAddress inetAddr : resolvedInetAddrs.addresses) {
        	step2. 将每个 InetAddress 格式的IP地址包装为 EquivalentAddressGroup 对象
        	servers.add(new EquivalentAddressGroup(new InetSocketAddress(inetAddr, port)));
        }
        // 跳过balancerAddresses和TXT的处理
        // step3. 通知listner，有数据更新
        savedListener.onAddresses(servers, attrs.build());
    }
}
```

主流程就是上面注释中的3个步骤：

1. 解析地址
2. 包装格式
3. 通知listener

注意这个工作是在异步线程中进行的，无法直接return结果，只能通过listener。

再继续看错误流程，如果解析地址失败：

```java
try {
    resolvedInetAddrs = delegateResolver.resolve(host);
} catch (Exception e) {
    // 遇到无法解析的情况
    synchronized (DnsNameResolver.this) {
        if (shutdown) {
            // 如果此时已经要求 shutdown，则不用继续处理，直接return
            return;
        }
        // 因为在生产中 timerService 是一个单线程的 GrpcUtil.TIMER_SERVICE
        // 我们需要将这个阻塞的工作交给 executor
        resolutionTask =
        timerService.schedule(new LogExceptionRunnable(resolutionRunnableOnExecutor), 1, TimeUnit.MINUTES);
    }
    // 通知 listener 遇到错误
    savedListener.onError(Status.UNAVAILABLE.withDescription(
    	"Unable to resolve host " + host).withCause(e));
    return;
}
```

上面的代码，在处理解析失败时，做了两个事情：

1. 通知 listener 出错了
2. 安排了一个 resolutionTask

出错了通知 listener 这个容易理解，resolutionTask 是做什么呢？ 我们细看 resolutionTask 相关的代码：

```java
// resolutionTask 的定义，一个标准的 ScheduledFuture
private ScheduledFuture< ? > resolutionTask;
......

// 唯一一个赋值的地方，就是当解析出错时，也就是上面的处理
resolutionTask = timerService.schedule(new LogExceptionRunnable(resolutionRunnableOnExecutor), 1, TimeUnit.MINUTES);
```

给 timerService 提交一个 LogExceptionRunnable 的任务，要求延迟1分钟执行，然后将得到的 feature 保存为 resolutionTask 。这个 resolutionTask 在 shutdown()方法和 resolutionRunnable 的 run()方法中有细节处理。

先看看 LogExceptionRunnable 的实现：

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
      // 如果是RuntimeException或者Error，就直接原样抛出去
      MoreThrowables.throwIfUnchecked(t);
      // 否则就生成一个新的AssertionError抛出去
      throw new AssertionError(t);
    }
  }
}
```

只是简单的执行 task 并在出错时记录日志，而这里的 task 是 resolutionRunnableOnExecutor，所以关键还是看 resolutionRunnableOnExecutor 里面的实现内容：

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

现在错误流程流程就清晰了：

1. 在解析失败时，就会给 timerService 安排一个一分钟之后执行的任务 resolutionTask
2. 在 resolutionTask 中，将重新用 executor 跑一次 resolutionRunnable
3. 如果继续解析失败，则循环上述过程

即解析失败则每隔一分钟尝试一次，直到成功。

注意在 resolutionRunnable 中，每次发现 resolutionTask 存在就会先 cancel 掉它，然后置为null：

```java
if (resolutionTask != null) {
    resolutionTask.cancel(false);
    resolutionTask = null;
}
```

所以上述的循环，只是在每次解析失败时，一旦解析成功，就会跳出循环。
