类Channel
=========

## 类定义

Channel 是一个 abstract class，位于package "io.grpc", 在 grpc-core 这个jar包中。

```java
package io.grpc;

import javax.annotation.concurrent.ThreadSafe;

@ThreadSafe
public abstract class Channel {
	......
}
```

通过 @ThreadSafe 注解标明 Channel 是线程安全的。

## 方法定义

### ClientCall()方法

```java
public abstract <RequestT, ResponseT> ClientCall<RequestT, ResponseT> newCall(
      MethodDescriptor<RequestT, ResponseT> methodDescriptor, CallOptions callOptions);
```

构建一个用于远程操作的 ClientCall 对象，通过给定的 MethodDescriptor 来指定。返回的 ClientCall 对象不会触发任何远程行为，直到 ClientCall.start(ClientCall.Listener, Metadata) 方法被调用。


### authority()方法

```java
public abstract String authority();
```

这个 Channel 连接到的目的地的 authority。通常是以 "host:port" 格式。




