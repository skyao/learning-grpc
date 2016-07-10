类ManagedChannel
================

类ManagedChannel 在 Channel 的基础上提供生命周期管理的功能。

实际实现式就是添加了 shutdown()/shutdownNow() 方法用于关闭 Channel，isShutdown()/isTerminated() 方法用于检测 Channel 状态， 以及 awaitTermination() 方法用于等待关闭操作完成。

## 类定义

```java
package io.grpc;

public abstract class ManagedChannel extends Channel {}
```

依然是 abstract class，还继续在 package io.grpc中。

## 类方法












