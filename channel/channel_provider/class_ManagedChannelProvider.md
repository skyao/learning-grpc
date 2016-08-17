类ManagedChannelProvider
=======================

ManagedChannelProvider 是 managed channel 的提供者，用于 transport 的不可知消费。

ManagedChannelProvider 的实现类**不能**抛出(异常)。如果抛出了，可能打断类装载。如果因为实现特有的原因会发生异常，实现应该优雅的处理异常并从 isAvailable() 方法返回 false 。

> 注：以上说明，来自 ManagedChannelProvider 的 Javadoc。

## 类定义

```java
package io.grpc;

@Internal
public abstract class ManagedChannelProvider {}
```

## 静态初始化

上面Javadoc的描述中提到的"类不能抛出异常，否则会打断类装载"，说的应该就是这个静态初始化的操作：

```java
private static final ManagedChannelProvider provider
      = load(getCorrectClassLoader());
```

getCorrectClassLoader()方法先获取正确的ClassLoader:

```java
private static ClassLoader getCorrectClassLoader() {
    if (isAndroid()) {
      //对于android平台，特殊一些
      return ManagedChannelProvider.class.getClassLoader();
    }
    //其他情况，默认都是取当前线程的Context ClassLoader
    return Thread.currentThread().getContextClassLoader();
}
```

然后load()方法装载并选择ManagedChannelProvider的具体实现，这里有三个步骤：

1. 装载所有可能的候选者
2. 排除掉不可用的候选者
3. 选择优先级最高的候选者

load()方法的具体代码：

```java
static ManagedChannelProvider load(ClassLoader classLoader) {
    Iterable<ManagedChannelProvider> candidates;
    if (isAndroid()) {
      // 对于android平台，直接hard code候选者
      candidates = getCandidatesViaHardCoded(classLoader);
    } else {
      // 其他情况，通过JDK的ServiceLoader装载候选者
      candidates = getCandidatesViaServiceLoader(classLoader);
    }
    List<ManagedChannelProvider> list = new ArrayList<ManagedChannelProvider>();
    for (ManagedChannelProvider current : candidates) {
      // 排除掉不可用的候选者
      if (!current.isAvailable()) {
        continue;
      }
      list.add(current);
    }
    if (list.isEmpty()) {
      // 如果为空返回null
      return null;
    } else {
      // 返回优先级最高的候选者
      return Collections.max(list, new Comparator<ManagedChannelProvider>() {
        @Override
        public int compare(ManagedChannelProvider f1, ManagedChannelProvider f2) {
          return f1.priority() - f2.priority();
        }
      });
    }
}
```

逻辑很清晰，继续看其中的两个细节，看候选者是如何被装载出来的。

1. android平台：hard code 两个可能的实现 okhttp 和 netty，如果类装载就只能忽略

    ```java
      public static Iterable<ManagedChannelProvider> getCandidatesViaHardCoded(
          ClassLoader classLoader) {
        List<ManagedChannelProvider> list = new ArrayList<ManagedChannelProvider>();
        try {
          list.add(create(Class.forName("io.grpc.okhttp.OkHttpChannelProvider", true, classLoader)));
        } catch (ClassNotFoundException ex) {
          // ignore
        }
        try {
          list.add(create(Class.forName("io.grpc.netty.NettyChannelProvider", true, classLoader)));
        } catch (ClassNotFoundException ex) {
          // ignore
        }
        return list;
      }
    ```

2. 普通平台：标准的JDK ServiceLoader 方式

```java
public static Iterable<ManagedChannelProvider> getCandidatesViaServiceLoader(
  ClassLoader classLoader) {
	return ServiceLoader.load(ManagedChannelProvider.class, classLoader);
}
```

## 最重要的provider()方法

对于调用者来说，最重要的就是 provider()方法，因为通常调用者都是这样使用：

```java
ManagedChannelProvider provider = ManagedChannelProvider.provider();
......
```

provider()方法的实现很简单，判断一下静态变量 provider ，如果为空则抛出异常 ProviderNotFoundException，提示没有可用的 channel service provider：

```java
public static ManagedChannelProvider provider() {
    if (provider == null) {
      throw new ProviderNotFoundException("No functional channel service provider found. " + "Try adding a dependency on the grpc-okhttp or grpc-netty artifact");
    }
    return provider;
}
```

## 和装载相关的抽象方法

在load()方法中，每个装载到的候选者，都需要实现这两个方法以便调用。

1. isAvailable()

	用来判断这个provider是否可用，需要考虑当前环境。如果返回 false，则其他任何方法都不安全。

    ```java
    protected abstract boolean isAvailable();
    ```

	实际在 load() 方法中，所有isAvailable()返回false的候选者都被直接排除。

2. priority()

	优先级，每个provider可用的范围是从1到10,需要考虑当前环境。5被视为默认值，然后根据环境检测调整。优先级0 并不是暗示这个provider不工作，只是会排在最后。

    ```java
    protected abstract int priority();
    ```

## 创建ChannelBuilder的抽象方法

ManagedChannelProvider 的主要功能，还是在于创建适当的 ManagedChannelBuilder，在 ManagedChannelBuilder 中定义了两个抽象方法要求子类做具体实现：

1. builderForAddress()： 使用给定的host和端口创建新的builder

    ```java
    protected abstract ManagedChannelBuilder< ?> builderForAddress(String name, int port);
    ```

2. builderForTarget()： 使用给定的target URI来创建新的builder

    ```java
    protected abstract ManagedChannelBuilder< ?> builderForTarget(String target);
    ```
