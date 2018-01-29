# 类DirectAddressNameResolver

在类 AbstractManagedChannelImplBuilder 中有一个内部类，DirectAddressNameResolverFactory，这里实现了一个 NameResolver ，用于处理 DirectAddress。

## Factory的代码

```java
private static final String DIRECT_ADDRESS_SCHEME = "directaddress";

private static class DirectAddressNameResolverFactory extends NameResolver.Factory {
    final SocketAddress address;
    final String authority;

	// 构造函数中直接提供 address 和 authority
    DirectAddressNameResolverFactory(SocketAddress address, String authority) {
      this.address = address;
      this.authority = authority;
    }

    @Override
    public NameResolver newNameResolver(URI notUsedUri, Attributes params) {
      return new NameResolver() {
         ......// 细节后面看
      }
    }

	@Override
    public String getDefaultScheme() {
      // 默认的scheme是固定的 "directaddress"
      return DIRECT_ADDRESS_SCHEME;
    }
}
```

## NameResolver的实现

DirectAddressNameResolverFactory的NameResolver的实现是一个内部匿名类：

```java
public NameResolver newNameResolver(URI notUsedUri, Attributes params) {
  return new NameResolver() {
    @Override
    public String getServiceAuthority() {
      // 直接返回factory中传入并保存的authority
      return authority;
    }

    @Override
    public void start(final Listener listener) {
      // 不用解析，直接将 factory 中传入并保存的 address 给出去
      listener.onAddresses(
              Collections.singletonList(new EquivalentAddressGroup(address)),
              Attributes.EMPTY);
    }

    @Override
    public void shutdown() {}
  };
}
```

## 总结

这个实现够简单，不过考虑到平时也是有需要用到直接地址的。


