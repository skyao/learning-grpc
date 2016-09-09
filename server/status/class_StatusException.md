类StatusException
=================

gRPC中定义了两个和 Status 相关的异常，分别是 StatusException 和 StatusRuntimeException。以异常的实行来表示并传递状态信息。

# 类StatusException

代码足够简单，继承自Exception，通过传递一个 Status 对象实例来构建，并从Status对象实例中获取异常信息。

```java
public class StatusException extends Exception {
  private static final long serialVersionUID = -660954903976144640L;
  private final Status status;
  private final Metadata trailers;

  public StatusException(Status status) {
    this(status, null);
  }

  @ExperimentalApi
  // 使用状态和跟踪元数据来构建 exception
  // 这个方法是 1.0.0-pre2 之后才增加的
  public StatusException(Status status, @Nullable Metadata trailers) {
    super(Status.formatThrowableMessage(status), status.getCause());
    this.status = status;
    this.trailers = trailers;
  }

  public final Status getStatus() {
    return status;
  }
}
```

Status.formatThrowableMessage()方法用于从 Status 中获取异常的 message 信息：

1. 异常的message

	 通过 Status 的formatThrowableMessage()方法从Status中得到message。

     formatThrowableMessage()方法的具体实现代码：

     ```java
    static String formatThrowableMessage(Status status) {
        if (status.description == null) {
        	// 如果没有description，则直接取code.toString()
            // code是一个枚举，code.toString()方法里面取的是枚举的name属性
            // 也就是 INVALID_ARGUMENT 之类的字符串
        	return status.code.toString();
        } else {
        	// 如果有description，则将code(同上实际是枚举的name)和description拼起来
        	return status.code + ": " + status.description;
        }
    }
     ```

2. 异常的casue：直接取Status对象的casue

# 类StatusRuntimeException

代码和实现与StatusException完全一致，只是继承的是RuntimeException。


