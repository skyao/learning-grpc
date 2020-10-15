# NameResolver的用法

## 使用场合

### 类ManagedChannelBuilder

类ManagedChannelBuilder 的 nameResolverFactory() 方法用于在构建 channel 时指定需要的 resolverFactory ：

```java
@ExperimentalApi
public abstract T nameResolverFactory(NameResolver.Factory resolverFactory);
```

为channel提供一个定制的 NameResolver.Factory。如果这个方法没有被调用，builder将在全局解析器注册(global resolver registry)中为提供的目标寻找一个工厂。

## 工作方式

NameResolver的工作方式，和 Channel / Transport / Load Balancer 有密切关系，详细细节请见后面的章节：

- [name resolver @ ManagedChannelImpl](../../channel/channel/impl/name_resolver.md)