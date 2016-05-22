类Status
========

> 注：下面内容翻译自 类Status 的 [javadoc](https://github.com/grpc/grpc-java/blob/master/core/src/main/java/io/grpc/Status.java)

通过提供标准状态码并结合可选的描述性的信息来定义操作的状态。状态的实例创建是通过以合适的状态码开头并补充额外信息：Status.NOT_FOUND.withDescription("Could not find 'important_file.txt'");

对于客户端，每个远程调用在完成时都将返回一个状态。如果发生错误，这个状态将以RuntimeException的方式传播到阻塞桩（blocking stubs），或者作为一个明确的参数给到监听器。

同样的，服务器可以通过抛出 StatusRuntimeException 或者 给回掉传递一个状态来报告状态。

提供工具方法来转换状态到exception并从exception中解析出状态。

# 类Status.Code

## 状态码定义

在Status类中，通过内部类 Status.Code 以标准java枚举的方式定义了状态码：

```java
public final class Status {
    public enum Code {
        OK(0),
        CANCELLED(1),
        ......
        UNAUTHENTICATED(16);
    }
}
```

这里定了17个（OK和其他16个错误）状态码，它们的详细定义和描述请见 [状态码详细定义](status_code_definition.md).

## 状态码的属性

Status.Code 有两个属性，类型为整型的value和对应的字符串表示方式的valueAscii：

```java
public enum Code {
	......
    private final int value;
    private final String valueAscii;

    private Code(int value) {
      this.value = value;
      this.valueAscii = Integer.toString(value);
    }

    public int value() {
      return value;
    }

    private String valueAscii() {
      return valueAscii;
    }
}
```

# 类Status

## STATUS_LIST

静态变量 STATUS_LIST 保存了所有的 Status 对象，对应每个 Status.Code 枚举定义：

```java
  // Create the canonical list of Status instances indexed by their code values.
  private static final List<Status> STATUS_LIST = buildStatusList();

  private static List<Status> buildStatusList() {
    TreeMap<Integer, Status> canonicalizer = new TreeMap<Integer, Status>();
    // 将所有的Code的枚举定义都游历
    for (Code code : Code.values()) {
      Status replaced = canonicalizer.put(code.value(), new Status(code));
      if (replaced != null) {
      	//去重处理，有些奇怪这里可能重复吗？输入的可是枚举定义，value按说不会定义错
        throw new IllegalStateException("Code value duplication between "
            + replaced.getCode().name() + " & " + code.name());
      }
    }
    return Collections.unmodifiableList(new ArrayList<Status>(canonicalizer.values()));
  }
```

## Code的status()方法

Code的status()方法通过使用 STATUS_LIST 来直接返回该状态码对应的 Status 对象：

```java
private Status status() {
	return STATUS_LIST.get(value);
}
```

直接用value做下标，因此枚举定义的value就是从0开始。

## Status的静态实例

Status中为每个 Status.Code 定义了一对一的静态的Status实例：

```java
public static final Status OK = Code.OK.status();
public static final Status CANCELLED = Code.CANCELLED.status();
......
public static final Status DATA_LOSS = Code.DATA_LOSS.status();
```

注意 Code.status() 方法是调 `STATUS_LIST.get(value)` 来获取 Status 实例。

## 构造 Status 对象

静态方法fromCodeValue()根据给定的状态码构建Status对象：

```java
public static Status fromCodeValue(int codeValue) {
    if (codeValue < 0 || codeValue > STATUS_LIST.size()) {
    	return UNKNOWN.withDescription("Unknown code " + codeValue);
    } else {
    	return STATUS_LIST.get(codeValue);
    }
}
```

对于有效的code值(0到STATUS_LIST.size())，直接返回对应的保存在 STATUS_LIST 中的实例，这样得到的实例和前面的静态定义实际是同一个实例。






























