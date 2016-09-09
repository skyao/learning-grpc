# 类Metadata

提供对读取和写入元数据数值的访问，元数据数值在调用期间交换。

这个类不是线程安全，实现应该保证header的读取和写入不在多个线程中并发发生。

## 类定义

```java
package io.grpc;

@NotThreadSafe
public final class Metadata{}
```

## 类属性

```java
// 所有的值列表都可以加入。值列表不能为空。
// 使用 LinkedHashMap 来保证一致的顺序，用于测试
private final Map<String, List<MetadataEntry>> store = new LinkedHashMap<String, List<MetadataEntry>>();

// 存储在这个原数据中的header的数量
private int storeCount;

// 构造函数，当应用层想发送元数据时调用
public Metadata() {}

// 构造函数，当transport层收到二进制元数据时调用
public Metadata(byte[]... binaryValues) {
	// 数据是以key/value这样一对一对出现，因此必须是2的整数
    checkArgument(binaryValues.length % 2 == 0,
        "Odd number of key-value pairs: %s", binaryValues.length);
    // 循环时注意i+=2,每两个两个处理
    for (int i = 0; i < binaryValues.length; i += 2) {
      // 第一个是name，作为key必要是 ASCII 字符集
      String name = new String(binaryValues[i], US_ASCII);
      // 第二个是value，通过 BINARY_HEADER_SUFFIX 来判断 header 是不是二进制
      storeAdd(name, new MetadataEntry(name.endsWith(BINARY_HEADER_SUFFIX), binaryValues[i + 1]));
    }
}

// 所有二进制 header 在他们的名字中应该有这个后缀。相反也是。
// 它的值是"-bin"。ASCII header 的名字一定不能以这个结尾。
public static final String BINARY_HEADER_SUFFIX = "-bin";
```

### 存取数据的方法

1. storeAdd(): 添加存储，注意这个方法真的是线程不安全！

	这个是private的内部方法，public的是put()方法。

	```java
    private void storeAdd(String name, MetadataEntry value) {
        List<MetadataEntry> values = store.get(name);
        if (values == null) {
            // 通常 header 的值是唯一的，因此选择很小的数组，初始值设置为1
            values = new ArrayList<MetadataEntry>(1);
            store.put(name, values);
        }
        // 数量加1
        storeCount++;
        // 支持多个值
        values.add(value);
    }
	```

2. headerCount(): 返回在这个元数据中的键值 header 的总数量，包含重复

	```java
    public int headerCount() {
    	return storeCount;
    }
	```

3. containsKey(): 如果给定的key有值则返回true，注意这里没有检查value是否可能为空列表

	```java
    public boolean containsKey(Key<?> key) {
    	return store.containsKey(key.name());
    }
	```

4. get(): 返回最后一个用给定name添加的元数据项，作为T解析，如果没有则返回null

	```java
    public < T> T get(Key< T> key) {
        List<MetadataEntry> values = store.get(key.name());
        if (values == null) {
          // 如果没有则返回null
          return null;
        }
        // values.size() - 1 是取最后一个，注意这里没有检查values是否可能为空列表
        MetadataEntry metadataEntry = values.get(values.size() - 1);
        return metadataEntry.getParsed(key);
    }
	```

5. getAll(): 返回所有用给定name添加的元数据项，作为T解析，如果没有则返回null。顺序和接收到的一致。

	```java
    public < T> Iterable< T> getAll(final Key< T> key) {
        if (containsKey(key)) {
          return Iterables.unmodifiableIterable(Iterables.transform(
              store.get(key.name()),
              new Function<MetadataEntry, T>() {
                @Override
                public T apply(MetadataEntry entry) {
                  return entry.getParsed(key);
                }
              }));
        }
        return null;
    }
	```

6. keys(): 返回存储中的所有key的不可变集合

	```java
    public Set< String> keys() {
    	return Collections.unmodifiableSet(store.keySet());
    }
	```

7. put(): 添加键值对。如果key已经有值，值被添加到最后。同一个key的重复的值是容许的。

	```java
    public < T> void put(Key< T> key, T value) {
        Preconditions.checkNotNull(key, "key");
        Preconditions.checkNotNull(value, "value");
        storeAdd(key.name, new MetadataEntry(key, value));
    }
	```

8. remove(): 删除key的第一个出现的value，如果删除成功返回true，如果没有找到返回false

	```java
    public < T> boolean remove(Key< T> key, T value) {
        Preconditions.checkNotNull(key, "key");
        Preconditions.checkNotNull(value, "value");
        List<MetadataEntry> values = store.get(key.name());
        if (values == null) {
          // 如果没有找到返回false
          return false;
        }
        // for循环是从第一个开始，也就是从添加最早的开始
        for (int i = 0; i < values.size(); i++) {
          MetadataEntry entry = values.get(i);
          if (!value.equals(entry.getParsed(key))) {
            // 检查每个项目的value
            continue;
          }
          values.remove(i);
          storeCount--;
          return true;
        }
        return false;
    }
	```

9. removeAll(): 删除给定key的所有值并返回这些被删除的值。如果没有找到值，则返回null

	```java
    public <T> Iterable<T> removeAll(final Key<T> key) {
        List<MetadataEntry> values = store.remove(key.name());
        if (values == null) {
          // 如果没有找到值，则返回null
          return null;
        }
        storeCount -= values.size();
        return Iterables.transform(values, new Function<MetadataEntry, T>() {
          @Override
          public T apply(MetadataEntry metadataEntry) {
            return metadataEntry.getParsed(key);
          }
        });
    }
	```

## 内部定义的类

### Marshaller

定义了两个 Marshaller 的interface，分别用于处理二进制和ASCII字符的值：

1. BinaryMarshaller： 用于序列化为原始二进制的元数据值的装配器
2. AsciiMarshaller: 用于序列化为 ASCII 字符的元数据值的装配器，值只包含下列字符：

	- 空格：0x20，但是不能在值的开头或者末尾。开始或者末尾的空白字符可能被去除。
	- ASCII 可见字符 (0x21-0x7E)

然后实现了这两个接口的内部匿名类：

```java
public static final BinaryMarshaller<byte[]> BINARY_BYTE_MARSHALLER = new BinaryMarshaller<byte[]>() {};

public static final AsciiMarshaller<String> ASCII_STRING_MARSHALLER = new AsciiMarshaller<String>() {};

static final AsciiMarshaller<Integer> INTEGER_MARSHALLER = new AsciiMarshaller<Integer>() {};
```

## 内部类 Metadata.Key

元数据项的key。考虑到元数据的解析和序列化。

### key名称中的有效字符

在key的名字中仅容许下列ASCII字符：

- 数字： 0-9
- 大写字符： A-Z（正常化到小写)
- 小写字符： a-z
- 特殊字符： -_.

这是 HTTP 字段名规则的一个严格子集。应用不应该发送或者接收带有非法key名称的元数据。当然，gRPC 类库可能加工任何接收到的元数据，即使它不遵守上面的限制。此外，如果元数据包含不遵守的字段名，他们也将被发送。在这种方式下，未知元数据字段被解析，序列化和加工，但是从不中断。他们类似protobuf的未知字段。

注意，有一个有效的 HTTP/2 token字符的子集，被定义在 RFC7230 3.2.6 节和 RFC5234 B.1节。

- <a href="https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md">Wire Spec</a>
- <a href="https://tools.ietf.org/html/rfc7230#section-3.2.6">RFC7230</a>
- <a href="https://tools.ietf.org/html/rfc5234#appendix-B.1">RFC5234</a>


```java
// 字段名的有效字符被定义在 RFC7230 和 RFC5234
private static final BitSet VALID_T_CHARS = generateValidTChars();

//在这里看到，仅有三个特殊字符，数字和小写字母是有效的
private static BitSet generateValidTChars() {
  BitSet valid  = new BitSet(0x7f);
  valid.set('-');
  valid.set('_');
  valid.set('.');
  for (char c = '0'; c <= '9'; c++) {
    valid.set(c);
  }
  // 仅在正常化之后验证，因此排除大小字母
  for (char c = 'a'; c <= 'z'; c++) {
    valid.set(c);
  }
  return valid;
}
```

validateName()方法用于验证输入的字符串(会先转为小写)是否有效：

```java
private static String validateName(String n) {
  checkNotNull(n);
  checkArgument(n.length() != 0, "token must have at least 1 tchar");
  for (int i = 0; i < n.length(); i++) {
    char tChar = n.charAt(i);
    // TODO(notcarl): remove this hack once pseudo headers are properly handled
    if (tChar == ':' && i == 0) {
      continue;
    }

    checkArgument(VALID_T_CHARS.get(tChar),
        "Invalid character '%s' in key name '%s'", tChar, n);
  }
  return n;
}
```

### Key的类定义

```java
public abstract static class Key<T> {}
```

这是一个abstract类，后面将看到它的几个子类实现。

### Key的属性和构造函数

```java
private final String originalName;
private final String name;
private final byte[] nameBytes;

private Key(String name) {
  // 原始名字是构造函数中传递过来的原始输入
  this.originalName = checkNotNull(name);
  // Intern the result for faster string identity checking.
  // 有效名字是经过处理并验证有效的名字：这里的处理就是简单的改为小写
  this.name = validateName(this.originalName.toLowerCase(Locale.ROOT)).intern();
  // nameBytes是有效名字的字节数组
  this.nameBytes = this.name.getBytes(US_ASCII);
}
```

### Key的抽象方法

定义了两个方法用来做元数据和字节数组之间的转换和解析：

```java
// 将元数据序列化到byte数组
abstract byte[] toBytes(T value);

// 从byte数组解析被序列化的元数据值
abstract T parseBytes(byte[] serialized);
```

### Key的子类实现

1. BinaryKey

    ```java
    private static class BinaryKey<T> extends Key<T> {
        private final BinaryMarshaller<T> marshaller;

        // 构造函数传入一个BinaryMarshaller
        private BinaryKey(String name, BinaryMarshaller<T> marshaller) {
		  ......
          this.marshaller = checkNotNull(marshaller, "marshaller is null");
        }

        @Override
        byte[] toBytes(T value) {
          // 使用这个BinaryMarshaller来转换 Key
          return marshaller.toBytes(value);
        }

        @Override
        T parseBytes(byte[] serialized) {
          // 使用这个BinaryMarshaller来解析 Key
          return marshaller.parseBytes(serialized);
        }
    }
    ```

2. AsciiKey： 类似，这次使用的是 AsciiMarshaller



