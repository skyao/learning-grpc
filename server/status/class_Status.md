类Status
========

> 注：下面内容翻译自 类Status 的 [javadoc](https://github.com/grpc/grpc-java/blob/master/core/src/main/java/io/grpc/Status.java)

通过提供标准状态码并结合可选的描述性的信息来定义操作的状态。状态的实例创建是通过以合适的状态码开头并补充额外信息：Status.NOT_FOUND.withDescription("Could not find 'important_file.txt'");

对于客户端，每个远程调用在完成时都将返回一个状态。如果发生错误，这个状态将以RuntimeException的方式传播到阻塞桩（blocking stubs），或者作为一个明确的参数给到监听器。

同样的，服务器可以通过抛出 StatusRuntimeException 或者 给回掉传递一个状态来报告状态。

提供工具方法来转换状态到exception并从exception中解析出状态。

## 类Status.Code

### 状态码定义

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

### 状态码的属性

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

### Code的status()方法

类Status.Code的status()方法通过使用 STATUS_LIST 来直接返回该状态码对应的 Status 对象：

```java
private Status status() {
	return STATUS_LIST.get(value);
}
```

## 类Status

### STATUS_LIST

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

直接用value做下标，因此枚举定义的value就是从0开始。

### Status的静态实例

Status中为每个 Status.Code 定义了一对一的静态的Status实例：

```java
public static final Status OK = Code.OK.status();
public static final Status CANCELLED = Code.CANCELLED.status();
......
public static final Status DATA_LOSS = Code.DATA_LOSS.status();
```

注意 Code.status() 方法是调 `STATUS_LIST.get(value)` 来获取 Status 实例。

### Status的属性和构造函数

Status的属性，注意都是final不可变：

```java
// code 用来保存状态码
private final Code code;
// description 是状态描述信息
private final String description;
// cause 是关联的异常
private final Throwable cause;


private Status(Code code) {
	// description 和 cause 都可以为null
	this(code, null, null);
}

private Status(Code code, @Nullable String description, @Nullable Throwable cause) {
    this.code = checkNotNull(code);
    this.description = description;
    this.cause = cause;
}
```

注意Status是注明 `@Immutable` 的，而且这个类是 final，不可以继承：

```java
@Immutable
public final class Status {}
```

### 构造 Status 对象的静态方法

1. 静态方法fromCodeValue()根据给定的状态码构建Status对象：

    ```java
    public static Status fromCodeValue(int codeValue) {
        if (codeValue < 0 || codeValue > STATUS_LIST.size()) {
            return UNKNOWN.withDescription("Unknown code " + codeValue);
        } else {
            return STATUS_LIST.get(codeValue);
        }
    }
    ```

    对于有效的 code 值(0到 STATUS_LIST.size())，直接返回对应的保存在 STATUS_LIST 中的实例，这样得到的实例和前面的静态定义实际是同一个实例。

    注意：返回的 Status 实例中 description 和 cause 属性都是null。

2. 静态方法 fromCode() 根据给定的Code对象构建Status对象：

    ```java
    public static Status fromCode(Code code) {
    	return code.toStatus();
    }

    //而toStatus()的实现非常简单，直接从 STATUS_LIST 里面取值。
    public enum Code {
        public Status toStatus() {
          return STATUS_LIST.get(value);
        }
    }
    ```

3. 静态方法 fromThrowable() 根据给定的 Throwable 对象构建Status对象：

    ```java
    public static Status fromThrowable(Throwable t) {
        Throwable cause = checkNotNull(t);
        // 循环渐进，逐层检查
        while (cause != null) {
          if (cause instanceof StatusException) {
            //StatusException就直接取status属性
            return ((StatusException) cause).getStatus();
          } else if (cause instanceof StatusRuntimeException) {
            //StatusRuntimeException也是直接取status属性
            return ((StatusRuntimeException) cause).getStatus();
          }
          //不是的话就继续检查cause
          cause = cause.getCause();
        }

        //最后如果还是找不到任何Status，就只能给 UNKNOWN
        return UNKNOWN.withCause(t);
    }
	```

### 修改 Status 对象的属性

由于 Status 对象是不可变的，因此如果需要修改属性，只能重新构建一个新的实例。

1. code 属性通常不需要修改
2. 设置 cause 属性

	```java
    // 使用给定 cause 创建派生的 Status 实例。
    // 但是不管如何，cause 不会从服务器传递到客户端
    public Status withCause(Throwable cause) {
        if (Objects.equal(this.cause, cause)) {
          return this;
        }
        return new Status(this.code, this.description, cause);
    }
	```

3. 设置 description 属性

	```java
    // 使用给定 description 创建派生的 Status 实例。
    public Status withDescription(String description) {
        if (Objects.equal(this.description, description)) {
        return this;
        }
        return new Status(this.code, description, this.cause);
    }
	```

	也可以在现有的 description 属性基础上增加额外的信息：

    ```java
    // 在现有 description 的基础上增加额外细节来创建派生的 Status 实例。
    public Status augmentDescription(String additionalDetail) {
        if (additionalDetail == null) {
          return this;
        } else if (this.description == null) {
          // 如果原来 description 为null，直接设置
          return new Status(this.code, additionalDetail, this.cause);
        } else {
          // 如果原来 description 不为null，通过"\n"连接起来
          return new Status(this.code, this.description + "\n" + additionalDetail, this.cause);
        }
    }
	```

### Status 的方法

1. isOk() 简单判断是否OK

    ```java
    public boolean isOk() {
        return Code.OK == code;
    }
    ```

2. asRuntimeException() 方法将当前 Status 对象转为一个 RuntimeException

    ```java
    public StatusRuntimeException asRuntimeException() {
    	return new StatusRuntimeException(this);
    }
    ```

	注意这里得到的 StatusRuntimeException 对象携带了当前 Status ，可以通过方法 fromThrowable() 找回这个 Status 实例。

3. 带跟踪元数据的 asRuntimeException() 方法将当前 Status 对象转为一个 RuntimeException并携带跟踪元数据

    ```java
    public StatusRuntimeException asRuntimeException(Metadata trailers) {
    	return new StatusRuntimeException(this, trailers);
    }
    ```

4. asException()功能类似，但是得到的是 Exception














