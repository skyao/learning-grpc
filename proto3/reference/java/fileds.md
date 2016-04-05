字段
=========

> 注：内容翻译自官网文档 [Fields](https://developers.google.com/protocol-buffers/docs/reference/java-generated#fields)

除了在前一节描述的方法之外，protocol buffer编译器为在.proto文件中定义的消息的每个字段生成访问器方法集合。读取字段的方法被定义在消息类和它对应的builder上，修改值的方法仅在builder上被定义。

注意方法名称通常使用驼峰法命名，就是在.proto文件中的字段名使用带下划线的小写(应该如此)。转换工作如下：

1. 对于名字中的每个下划线，下划线被删除，而下一个字母大写。
2. 如果名字有附加前缀(如 "get")，第一个字母大写，否则小写

因此，字段 `foo_bar_baz` 变成 `fooBarBaz`. 如果带 `get` 前缀, 将是 `getFooBarBaz`.

除访问方法之外，编译器为每个字段生成一个包含它的字段编号的整型常量。常量名是转换为大写并加上 `_FIELD_NUMBER` 字段名字。例如，假设字段  `optional int32 foo_bar = 5`。编译器将生成常量 `public static final int FOO_BAR_FIELD_NUMBER = 5;`。

## Singular Fields (proto2)

对于这些字段定义的任何一个：

```bash
optional int32 foo = 1;
required int32 foo = 1;
```

编译器将在消息类和它的builder中生成下列访问器方法：

- `boolean hasFoo()`: 如果字段被设置返回true
- `int getFoo()`: 返回字段的当前值。如果字段没有设置，返回默认值。

编译器将仅在消息的builder中生成下列方法：

- `Builder setFoo(int value)`: 设置字段的值。调用之后，hasFoo()将返回true而getFoo()将返回这个值。
- `Builder clearFoo()`: 清除字段的值。调用之后，hasFoo() 将返回 false 而 getFoo() 将返回默认值。

对于其他简单字段类型，将根据 [scalar value types table](https://developers.google.com/protocol-buffers/docs/proto#scalar) 选择合适的java类型。对于消息和枚举类型，值类型被消息或者枚举类代替。

### 内嵌消息字段

对于消息类型，setFoo() 方法也接受消息builder类型的实例作为参数。这仅仅是一个快捷方式，等价于调用 builder的.build()方法并将结果传递给方法。

如果字段没有设置， getFoo() 方法将返回一个任何字段都没有设置的 Foo 实例(很可能是Foo.getDefaultInstance()返回的实例)。

此外，编译器生成两个访问器方法容许你访问消息类型关联的subbuilder。下列方法将被消息类和它的builder中生成：

- `FooOrBuilder getFooOrBuilder()`: 如果字段的builder已经存在则返回字段的builder;如果不存在则返回消息

编译器仅在消息的builder上生成下列方法：

- `Builder getFooBuilder()`: 返回字段的builder

## Singular Fields (proto3)

对于这个字段定义：

```bash
int32 foo = 1;
```

编译器将在消息类和它的builder中生成下列访问器方法：

- `int getFoo()`: 返回字段的当前值。如果字段没有设置，返回默认值。

编译器将仅在消息的builder中生成下列方法：

- `Builder setFoo(int value)`: 设置字段的值。调用之后，hasFoo()将返回true而getFoo()将返回这个值。
- `Builder clearFoo()`: 清除字段的值。调用之后，hasFoo() 将返回 false 而 getFoo() 将返回默认值。

对于其他简单字段类型，将根据 [scalar value types table](https://developers.google.com/protocol-buffers/docs/proto#scalar) 选择合适的java类型。对于消息和枚举类型，值类型被消息或者枚举类代替。

### 内嵌消息字段

对于消息字段类型，将在消息类和它的builder上生成一个额外的访问器方法:

- `boolean hasFoo()`: 如果字段已经设置则返回true
- `setFoo()` 也接受消息builder类型的实例作为参数。这仅仅是一个捷方式，等价于调用 builder 的.build()方法并将结果传递给方法。

如果字段没有设置， getFoo() 方法将返回一个任何字段都没有设置的 Foo 实例(很可能是Foo.getDefaultInstance()返回的实例)。

此外，编译器生成两个访问器方法容许你访问消息类型关联的subbuilder。下列方法将被消息类和它的builder中生成：

- `FooOrBuilder getFooOrBuilder()`: 如果字段的builder已经存在则返回字段的builder;如果不存在则返回消息

编译器仅在消息的builder上生成下列方法：

- `Builder getFooBuilder()`: 返回字段的builder

### 枚举字段

对于枚举字段类型，将会在消息类和它的builder上生成额外的访问器方法:

- `int getFooValue()`: 返回枚举的整型值

编译器仅在消息的builder上生成下列方法：

- `Builder setFooValue(int value)`: 设置枚举的整型值。

此外， 如果枚举类型未知，getFoo() 将返回 UNRECOGNIZED - 这是一个特别的附加值， 由proto3编译器添加到生成的枚举类型。

## 重复字段

对于这个字段定义：

```bash
repeated int32 foo = 1;
```language
```

编译器将在消息类和它的builder中生成下列访问器方法：

- `int getFooCount()`: 返回当前字段中的元素数量
- `int getFoo(int index)`: 返回给定的以0为基准的下标位置的元素。
- `List<Integer> getFooList()`: 以列表方式返回所有字段。如果字段没有设置，返回一个空列表。返回的列表是对于消息类不可变而对于消息builder类不可修改(原文：The returned list is immutable for message classes and unmodifiable for message builder classes.)。

编译器仅在消息的builder上生成下列方法：

- `Builder setFoo(int index, int value)`: 设置给定以0为基准的下标位置的元素的值
- `Builder addFoo(int value)`: 使用给定的值附加一个新元素到字段中(最后面)
- `Builder addAllFoo(Iterable<? extends Integer> value)`: 将给定 Iterable 中的所有元素都附加到字段中(最后面)
- `Builder clearFoo()`: 从字段中删除所有字段。调用这个方法之后，getFooCount()将返回0.

对于其他简单字段类型，将根据 [scalar value types table](https://developers.google.com/protocol-buffers/docs/proto#scalar) 选择合适的java类型。对于消息和枚举类型，值类型被消息或者枚举类代替。

### 重复内嵌消息字段

对于消息类型，setFoo() 和 addFoo() 也接受消息builder类型的一个实例作为参数。这仅仅是一个快捷方式，等价于调用 builder的.build()方法并将结果传递给方法。

此外，编译器将为消息类型在消息类和它的builer上生成两个额外的访问器方法，容许你访问关联的subbuilder：

- `FooOrBuilder getFooOrBuilder(int index)`: 如果特定元素的builder已经存在则返回builder;如果不存在则返回元素。如果这个方法调用来自消息类，它将总是返回消息而不是builder。
- `List<FooOrBuilder> getFooOrBuilderList()`: 以不可修改的builder列表(如果可用)的方式返回整个字段。如果这个方法调用来自消息类，它将总是返回消息的不可变列表而不是不可修改的builder列表。

编译器仅在消息的builder上生成下列方法：

- `Builder getFooBuilder(int index)`: 返回指定下标的元素的builder。
- `Builder addFooBuilder(int index)`: 为位于指定下标的默认消息实例附加并返回builder。
- `Builder addFooBuilder()`: 为默认消息实例附加并返回builder。
- `Builder removeFoo(int index)`: 删除位于给定以0为基准的下标位置的元素。
- `List<Builder> getFooBuilderList()`: 以不可修改的builder列表的方式返回整个字段

### 重复枚举字段 (仅用于proto3)

编译器将在消息类和它的builder上生成下列额外方法:

- `int getFooValue(int index)`: 返回位于指定下标的枚举的整型值
- `List getFooValueList()`: 以Integer列表的方式返回整个字段。

编译器将仅在消息的builder上生成下列额外方法：

- `Builder setFooValue(int index, int value)`: 设置位于指定下标的枚举的整型值。

## Oneof 字段

对于这个oneof字段定义:

```java
oneof oneof_name {
    int32 foo = 1;
    ...
}
```

编译器将在消息类和它的builder中生成下列访问器方法：

- boolean hasFoo() (仅用于proto2): 如果oneof实例是Foo则返回true。
- int getFoo(): 如果oneof实例是Foo则返回oneof_name的当前值。否则，返回这个字段的默认值。

编译器将仅在消息的builder中生成下列方法：

- Builder setFoo(int value): 设置 oneof_name 为这个值并设置oneof实例为FOO。调用这个方法之后， hasFoo()将返回true，getFoo()将返回值而getOneofNameCase()将返回FOO。
- Builder clearFoo():

	- 如果oneof实例不是FOO则不会有任何变化
	- 如果oneof实例是FOO，设置 oneof_name 为null，oneof实例为 ONEOF_NAME_NOT_SET。调用这个方法之后，hasFoo()将返回false，getFoo()将返回默认值而getOneofNameCase()将返回 ONEOF_NAME_NOT_SET 。

对于其他简单字段类型，将根据 [scalar value types table](https://developers.google.com/protocol-buffers/docs/proto#scalar) 选择合适的java类型。对于消息和枚举类型，值类型被消息或者枚举类代替。

## Map 字段

对于这个 map 字段定义:

```java
	map<int32, int32> weight = 1;
```

编译器将在消息类和它的builder中生成下列访问器方法：

- `Map<Integer, Integer> getWeight();`: 返回不可修改的map。

编译器将仅在消息的builder中生成下列方法：

- `Map<Integer, Integer> getMutableWeight();`: 返回可变的Map。注意对这个方法的多次调用将返回不同的map实例。返回的map引用可能对任何后续对builder的方法调用无效。
- `Builder putAllWeight(Map<Integer, Integer> value);`: 添加在给定map中的所有记录到这个字段。



