# InUseStateAggregator 的工作方式

上一节发现空闲模式的进入和退出是由 InUseStateAggregator 来进行控制的，继续看 InUseStateAggregator 是如何工作的。

## 类InUseStateAggregator

InUseStateAggregator，字面意思是"正在使用的状态聚合器"，javadoc如此描述：

> Aggregates the in-use state of a set of objects.
> 聚合一个对象集合的正在使用的状态

这是一个grpc的内部抽象类：

```java
package io.grpc.internal;
abstract class InUseStateAggregator<T> {}
```

### 实现原理

类InUseStateAggregator的实现原理很直白，就是每个要使用这个聚合器的调用者都要存进来一个对象(就是范型T)，然后用完之后再取出来，这样就可以通过保存对象的数量变化判断开始使用(从无到有)或者已经不再使用(从有到无):

```java
private final HashSet<T> inUseObjects = new HashSet<T>();

final void updateObjectInUse(T object, boolean inUse) {
    synchronized (getLock()) {
      // 记录更新前的原始数量
      int origSize = inUseObjects.size();
      if (inUse) {
        // inUse为true，表示增加正在使用的状态，保存对象
        inUseObjects.add(object);
        if (origSize == 0) {
          // 如果原始数量为0,表示从无到有，此时回调抽象方法 handleInUse()
          handleInUse();
        }
      } else {
        // inUse为false，表示检查正在使用的状态，删除对象
        boolean removed = inUseObjects.remove(object);
        if (removed && origSize == 1) {
          // 如果删除成功并且原始使用为1,表示从有到无，此时回调抽象方法 handleNotInUse()
          handleNotInUse();
        }
      }
    }
}
```

isInUse() 函数通过检查是否有保存的对象来判断是否正在使用：

```java
final boolean isInUse() {
    synchronized (getLock()) {
      return !inUseObjects.isEmpty();
    }
}
```

### 状态变更回调

当状态变更时，回调这两个抽象方法：

1. handleInUse()： 当被聚合的使用中的状态被修改为 true 时调用，这意味着至少一个对象正在使用中。

    ```java
    abstract void handleInUse();
    ```

2. handleNotInUse()： 当被聚合的使用中的状态被修改为 false 时调用，这意味着没有对象正在使用中。

    ```java
    abstract void handleNotInUse();
    ```

## ManagedChannelImpl中的实现

### 回顾 InUseStateAggregator 定义

回顾 ManagedChannelImpl 中对 inUseStateAggregator 的定义：

```java
final InUseStateAggregator<Object> inUseStateAggregator =
    new InUseStateAggregator<Object>() {
	......
    void handleInUse() {
      // 当状态变更为使用中时，退出空闲模式
      exitIdleMode();
    }

    void handleNotInUse() {
      // 当状态变更为没有人使用时，重新安排空闲timer以便在稍后进入空闲模式
      rescheduleIdleTimer();
    }
};
```

### 调用 InUseStateAggregator

在ManagedChannelImpl中，当 transport 状态变更时就会调用 InUseStateAggregator：

```java
final TransportManager<ClientTransport> tm = new TransportManager<ClientTransport>() {
	......
	ts = new TransportSet(......, new TransportSet.Callback() {
    	......
        public void onInUse(TransportSet ts) {
        	// 当 transportset 的状态变更为 InUse 时
        	inUseStateAggregator.updateObjectInUse(ts, true);
        }

        public void onNotInUse(TransportSet ts) {
        	// 当 transportset 的状态变更为 NotInUse 时
        	inUseStateAggregator.updateObjectInUse(ts, false);
        }
	}
}
```

还有 InterimTransportImpl:

```java
private class InterimTransportImpl implements InterimTransport<ClientTransport> {
    public void transportTerminated() {
    	......
    	inUseStateAggregator.updateObjectInUse(delayedTransport, false);
    }

    @Override public void transportInUse(boolean inUse) {
    	inUseStateAggregator.updateObjectInUse(delayedTransport, inUse);
    }
}
```

> 注： Transport 的详细内容不再继续展开了。