# 类 NameResolver

## 类定义

NameResolver 定义为一个 abstract 类，而不是 interface ：

```java
public abstract class NameResolver {}
```

## 类方法

#### getServiceAuthority()

返回用于验证与服务器的连接的权限(authority)。 必须来自受信任的来源，因为如果权限被篡改，RPC可能被发送到攻击者，泄露敏感用户数据。

实现必须以不阻塞的方式生成它，通常在一行中(in line)，必须保持不变。使用同样的参数从同一个的 factory 中创建出来的 NameResolver 必须返回相同的 authority 。

```java
public abstract String getServiceAuthority();
```

#### start()

开始解析。listener 用于接收目标的更新。

```java
public abstract void start(Listener listener);
```

#### shutdown()

停止解析。listener 的更新将会停止。

```java
public abstract void shutdown();
```

#### refresh()

重新解析名字。

只能在 start() 方法被调用之后调用。

这里仅仅是一个暗示。实现类将它作为一个信号，但是可能不会立即开始解析。它绝不抛出异常。

默认实现是什么都不做。

```java
public void refresh() {}
```

## 内部类 Factory

创建 NameResolver 实例的工厂类。

```java
public abstract static class Factory {
	/**
     * 端口号，用于当目标或者底层命名系统没有提供端口号的情况
     */
    public static final Attributes.Key<Integer> PARAMS_DEFAULT_PORT =
        Attributes.Key.of("params-default-port");
}
```

### newNameResolver()

创建 NameResolver 用于给定的目标URI，或者在给定URI无法被这个 factory 解析时返回 null。决定应该仅仅基于 URI 的 scheme。

参数 targetUri 表示要解析的目标 URI，而这个 URI 的 scheme 必须不能为 null。

参数 params 是可选参数。权威key在 Factory 中以 `PARAMS_*` 字段定义。

```java
public abstract NameResolver newNameResolver(URI targetUri, Attributes params);
```

### getDefaultScheme()

返回默认 scheme， 当 ManagedChannelBuilder.forTarget(String) 方法被给到 authority 字符串而不是符合的 URI时，用于构建 URI 。

```java
public abstract String getDefaultScheme();
```


## 内部接口 Listener

接收地址更新。

所有的方法都要求快速返回。

```java
public interface Listener {}
```

### onUpdate()

注意： 已经被废弃，改用 onAddresses() 方法。

```java
@Deprecated
void onUpdate(List<ResolvedServerInfoGroup> servers, Attributes attributes);
```

### onAddresses()

处理被解析的地址和配置的更新。

实现不可以修改给定的参数 servers 。

参数 servers 指被解析的服务器地址。空列表将触发 onError() 方法。

```java
void onAddresses(List<EquivalentAddressGroup> servers, Attributes attributes);
```

### onError()

处理从 resolver 而来的错误。

参数 error 为非正常的状态

```java
void onError(Status error);
```