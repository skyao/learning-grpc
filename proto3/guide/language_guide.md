Protocl Buffer 3 语言指南
========================

> 注: 内容翻译来自官网资料 [Language Guide (proto3)](https://developers.google.com/protocol-buffers/docs/proto3).

这份指南描述如何使用protocol buffer语言来构建你的protocol buffer数据,包括.proto文件语法和如何从.proto文件生成数据访问类. 覆盖protocol buffers语言的proto3版本, 对于老版本proto2的语法,请参考[Proto2语言指南](https://developers.google.com/protocol-buffers/docs/proto).

这是一个参考指南 - 如果需要逐步渐进的,使用到这份文档描述的众多特性的范例, 请见你所选语言的教程[教程](https://developers.google.com/protocol-buffers/docs/tutorials).

# 定义消息类型

首先我们来看一个非常简单的例子. 让我们假设你想定义一个搜索请求消息格式, 每个搜索请求有一个查询字符串, 搜索结果的第几页/每页结果数量. 这里是你用来定义消息类型的.proto文件:

```java
syntax = "proto3";

message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}
```

1. 第一行指明当前使用的是proto3语法, 如果不指定则默认为proto2. **必须是.proto文件的除空行和注释内容之外的第一行**
2. SearchRequest 消息定义指明了3个字段. 每个字段有名字和类型.

## 定义字段类型

上面例子中, 所有字段都是Scalar Types/简单类型(下面会详细讲述), 包括两个int和一个string. 此外还可以支持复合类型包括枚举类型.

## 指定标签

可以看到消息定义的每个字段都有一个唯一的数字型标签. 这个标签用于在消息的二进制格式中标识字段, 一旦消息类型被使用后不可以再修改.

注意标签的值在1和15之间时编码只需一个字节, 包括标识值和字段类型(可以在Protocol Buffer Encoding中找到更多信息). 标签在16到2047之间将占用两个字节. 因此应该将从1到15的标签分派给最频繁出现的消息元素. 记得保留一些空间给未来可能添加的频繁出现的元素.

可以指定的最小的标签值是1, 最大的是2的29次方-1, 即536870911. 另外19000到19999(FieldDescriptor::kFirstReservedNumber through FieldDescriptor::kLastReservedNumber)不能使用, 因为Protocol Buffers的实现自己保留这个标签段.

## 指定字段规则

消息字段可以是下列之一:

1. 单数: 一个定义良好的消息可以有0个或1个此字段(但是不能超过1个).
2. 重复: 这个字段可以在定义良好的消息中重复任意次(包括0次).重复值的顺序将被维持原状.

由于历史原因, 简单数字类型的重复字段并没有编码为最有效率的方式. 新的代码应该使用特别选项[packed=true]来得到更有效率的编码. 例如:

	 repeated int32 samples = 4 [packed=true];

可以在Protocol Buffer Encoding中找到更多的关于packed encoding的信息.

## 添加更多消息类型

可以在单个 .proto 文件中定义多个消息类型. 适用于定义多个相关消息, 例如, 如果你想定义应答消息格式来相应你的SearchResponse消息类型,你应该将它添加到同一个.proto文件.

```java
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
}

message SearchResponse {
 ...
}
```

## 添加注释

可以使用C/C++风格的"//"语法在.proto文件中添加注释.

```java
message SearchRequest {
  string query = 1;
  int32 page_number = 2;  // Which page number do we want?
  int32 result_per_page = 3;  // Number of results to return per page.
}
```

## 保留字段

当你更新消息类型,需要彻底删除一个字段时, 或者注释掉它, 未来的用户在实现他们对这个类型的更新时可以重用这个标签数字. 这将导致言中的问题, 如果他们后来装载同一个.proto文件的老版本, 败落数据冲突, 隐私缺陷以及其他. 为了确保不发生这样的事情, 有一个方法时指明这个要删除的字段为保留字段. 如果未来任何用户试图私用这个字段标识符protocol buffer的编译器就是告警.

```java
message Foo {
  reserved 2, 15, 9 to 11;
  reserved "foo", "bar";
}
```

注意: **不能在同一个保留字段声明中混合使用名字和标签数字**

## .proto可以生成什么?

对.proto文件执行protocol buffer的编译器时, 编辑器会生成你所选语言的代码. 你可以用来处理你在文件中定义的消息类型, 包括读写字段值, 系列化消息到输出流, 和从输入流中解析消息.

- 对于c++, 编译器为每个.proto文件生成.h和.cc文件, 并为每个在文件中描述的消息类型生成一个类
- 对于Java, 编译器为每个消息类型生成一个.java文件, 还有用于创建消息实例的Builder类.
- Python小有不同 - Python编译器为.proto文件中的每个消息类型生成带有静态描述符的模块, 这些模块将和metaclass一起在运行时用来创建必须的Python数据访问类.
- 编译器生成一个.pb.go文件, .proto文件中的每个消息类型在这个文件中都有一个类型.
- 对于Ruby, 编译器生成一个.rb文件, 带用包含消息类型的Ruby模块.
- 对于JavaNano, 编译器输出类似java但是没有builder类.
- 对于Objective-C, 编译器从每个.proto文件生成一个pbobjc.h和pbobjc.m文件,文件中描述的每个消息类型都有一个类.
- 对于C#, 编译器从每个.proto文件生成一个.cs文件,文件中描述的每个消息类型都有一个类.

你可以通过所选语言的教程(proto3版本即将到来)来找到更多关于如何使用每个语言的API的信息. 对于API的细节, 请看相应的[API参考手册](https://developers.google.com/protocol-buffers/docs/reference/overview)(proto3版本即将到来).

# 简单(Scalar)值类型

注: scalar是标量的意思,但是在如果翻译为标量有点不知所云, 先翻译为简单类型试试.

A scalar message field can have one of the following types – the table shows the type specified in the .proto file, and the corresponding type in the automatically generated class:

简单消息字段可以有下列类型其中之一 - 下面的表格显示在.proto文件中指明的类型和在自动生成类中的相应类型.

注: 原表格太复杂, 翻译没有必要, 请见[原页面](https://developers.google.com/protocol-buffers/docs/proto3#scalar). 

下面是简化版本, 只列出了我关心的java类型映射:

| .proto Type   | Java Type     |
| ------------- |:-------------:|
| double        | double 		|
| float         | float      	|
| int32			| int 	        |
| int32			| int 	        |
| int64			| long 	        |
| bool			| boolean       |
| string		| String        |
| bytes			| ByteString    |

TBD: ByteString 是个什么东东?

你可以在[Protocol Buffer Encoding](https://developers.google.com/protocol-buffers/docs/encoding)中找到更多资料, 了解当你序列化消息的时候, 这些类型是如何编码的.

# 默认值

当消息被解析时, 如果被编码的消息没有包含特定的简单元素, 被解析的对象对应的字段被设置为默认值. 默认值是和类型有关的:

- 对于strings, 默认值是空字符串(注, 是"", 而不是null)
- 对于bytes, 默认值是空字节(注, 应该是byte[0], 注意这里也不是null)
- 对于boolean, 默认值是false.
- 对于数字类型, 默认值是0.
- 对于枚举, 默认值是第一个定义的枚举值, 而这个值必须是0.
- 对于消息字段, 默认值是null.

对于重复字段, 默认值是空(通常都是空列表)

注意: 对于简单字段, 当消息被解析后, 是没有办法知道这个字段到底是有设置值然后恰巧和默认值相同(例如一个boolean设置为false)还是这个字段没有没有设置值而取了默认值. 例如, 不要用一个boolean值然后当设置为false时来切换某些行为, 而你又不希望这个行为默认会发生. 同样请注意: 如果一个简单消息字段被设置为它的默认值, 这个值不会被序列化.

# 枚举

当你定义消息类型时, 你可能希望某个字段只能有预先定义的多个值中的一个. 例如, 假设你想为每个SearchRequest添加一个corpus字段, 而corpus可以是UNIVERSAL, WEB, IMAGES, LOCAL, NEWS, PRODUCTS 或 VIDEO. 你可以简单的添加一个枚举到消息定义, 为每个可能的值定义常量.

在下面的例子中, 我们添加一个名为Corpus的枚举类型, 定义好所有可能的值, 然后添加一个类型为Corpus的字段:

```java
message SearchRequest {
  string query = 1;
  int32 page_number = 2;
  int32 result_per_page = 3;
  enum Corpus {
    UNIVERSAL = 0;
    WEB = 1;
    IMAGES = 2;
    LOCAL = 3;
    NEWS = 4;
    PRODUCTS = 5;
    VIDEO = 6;
  }
  Corpus corpus = 4;
}
```

如你所见, Corpus 枚举的第一个常量设置到0: 每个枚举定义必须包含一个映射到0的常量作为它的第一个元素. 这是因为:

- 必须有一个0值, 这样我们才能用0来作为数值默认值.
- 0值必须是第一个元素, 兼容proto2语法,在proto2中默认值总是第一个枚举值

可以通过将相同值赋值给不同的枚举常量来定义别名. 为此需要设置allow_alias选项为true, 否则当发现别名时protocol编译器会生成错误消息.

```java
enum EnumAllowingAlias {
  option allow_alias = true;
  UNKNOWN = 0;
  STARTED = 1;
  RUNNING = 1;
}
enum EnumNotAllowingAlias {
  UNKNOWN = 0;
  STARTED = 1;
  // RUNNING = 1;  // Uncommenting this line will cause a compile error inside Google and a warning message outside.
}
```

枚举常量必须在32位整形的范围内. 由于枚举值使用varint encoding, 负值是效率低下的因此不推荐使用.你可以在消息定义中定义枚举, 如前面例子那样, 或者在外部 - 这些枚举可以在.proto文件的任意消息定义中重用. 你也可以用在一个消息中声明的枚举类型作为别的消息的字段类型, 需要使用语法MessageType.EnumType.

当你运行protocol buffer 编译器处理使用枚举的.proto文件时, 生成的代码将会有java或c++的对应枚举, 对于Python, 在生成的运行时类中会有一个特别的EnumDescriptor用于创建带有整型值的symbolic常量集合.

在反序列过程中, 未被识别的枚举值将被保留在消息中, 但是当消息被反序列号时将会如何表现是和语言有关的. 在支持开放枚举类型可以用定义范围之外的值的语言中, 例如C++和Go, 未知的枚举值被简单保存为它底层整型描述. 在封闭枚举类型的语言例如Java中, 一个枚举的特例用于表示这个未识别的值, 底层整型可以被特殊的访问器访问. 在其他案例中, 如果消息被系列化, 这个未识别的值将和消息一起被序列号.

注: 下面这段内容有点不能理解,翻译的不好, 以后理解清楚再来修订.

> In languages with closed enum types such as Java, a case in the enum is used to represent an unrecognized value, and the underlying integer can be accessed with special accessors. In either case, if the message is serialized the unrecognized value will still be serialized with the message.

关于如何在应用中处理消息枚举的更多信息, 请看所选语言的生成代码指南(generated code guide).

# 使用其他消息类型

可以使用其他消息类型作为字段类型. 例如, 假设你想在每个SearchResponse消息中包含Result消息 - 为了做到这点, 你可以在相同的.proto文件中定义Result消息类型然后具体指定SearchResponse中的一个字段为Result类型:

```java
message SearchResponse {
  repeated Result result = 1;
}

message Result {
  string url = 1;
  string title = 2;
  repeated string snippets = 3;
}
```

## 导入定义

在上面的例子中, Result消息类型是定义在和SearchResponse同一个文件中 - 如果你想要作为字段类型使用的消息类型已经在其他的.proto文件中定义了呢?

你可以通过导入来使用来自其他.proto文件的定义. 为了导入其他.proto的定义, 需要在文件的顶端增加导入声明:

> import "myproject/other_protos.proto";

默认只能从直接导入的.proto文件中使用其中的定义. 但是, 某些情况下可能需要移动.proto文件到新的位置. 相比直接移动.proto文件并更新所有的调用点, 现在可以有其他的方法: 在原有位置放置一个伪装的.proto文件, 通过使用import public方式转发所有的import到新的位置. 其他任何导入这个包含import public语句的proto文件都可以透明的得到通过import public方法导入的依赖. 例如:

```java
// new.proto
// 所有的依赖被移动到这里
```
```java
// old.proto
// 这里是所有client需要导入的proto
import public "new.proto";
import "other.proto";
```
```java
// client.proto
import "old.proto";
// 这里可以使用old.proto 和 new.proto 中的定义, 但是other.proto中的不能使用.
```

protocol编译器在通过命令行-I/--proto_path参数指定的目录集合中搜索导入的文件. 如果没有指定, 则在编译器被调用的目录下查找. 通常应该设置--proto_path参数到项目所在的根目录然后为所有的导入使用完整限定名.

## 使用proto2的消息类型

导入proto2的消息类型并在proto3消息中使用是可以的, 反之也如此. 但是, **proto2的枚举不能在proto3语法中使用**

# 内嵌类型

可以在消息类型内部定义和使用消息类型, 如下面的例子所示 - 这里Result消息被定义在SearchResponse消息内部:

```java
message SearchResponse {
  message Result {
    string url = 1;
    string title = 2;
    repeated string snippets = 3;
  }
  repeated Result result = 1;
}
```

如果想在父消息类型之外重用消息类型, 可以使用 Parent.Type 来引用:

```java
message SomeOtherMessage {
  SearchResponse.Result result = 1;
}
```

只要愿意可以内嵌的更深:

```java
message Outer {                  // Level 0
  message MiddleAA {  // Level 1
    message Inner {   // Level 2
      int64 ival = 1;
      bool  booly = 2;
    }
  }
  message MiddleBB {  // Level 1
    message Inner {   // Level 2
      int32 ival = 1;
      bool  booly = 2;
    }
  }
}
```

# 更新消息类型

如果一个已有的消息类型不再满足需要- 例如, 想增加额外的字段 - 但是依然希望原有格式创建的代码可以继续使用, 不用担心. 可以很简单的更新消息类型而不用打破现有的代码.仅仅需要记住下面的规则:

- 不要改动任何现有字段的数字标签
- 如果添加新的字段时, 使用"老"消息格式系列化后的任何消息都可以被新生成的代码解析. 你需要留意这些元素的默认值以便新的代码可以正确和老代码生成的消息交互. 类似的, 新代码创建的消息可以被老代码解析: 解析时新的字段被简单的忽略. 注意当消息反序列化时未知字段会失效, 因此如果消息被传递给新代码, 新的字段将不再存在(这个行为和proto2不同, 在proto2中未知字段会和消息一起序列化).
- 字段可以被删除, 但是要求在更新后的消息类型中原来的标签数字不再使用.可以考虑重命名这个字段, 或者添加前缀"OBSOLETE_", 或者保留标签, 以便你的.proto文件未来的用户不会不小心重用这个数字.
- int32, uint32, int64, uint64, 和 bool 是完全兼容的 – 这意味着可以将一个字段的类型从这些类型中的一个修改为另外一个而不会打破向前或者向后兼容.如果一个数字解析时不匹配相应的类型, 那么效果会和在c++里面做类型转换一样(例如64位数字被作为32位整型读取, 将被转换为32位).
- sint32 和 sint64 是彼此兼容的,但是和其他整型类型不兼容.
- string 和 bytes 是兼容的, 如果bytes是有效的UTF-8.
- 嵌入式的消息和bytes兼容, 如果bytes包含这个消息的编码后的内容.
- fixed32 兼容 sfixed32, 而 fixed64 兼容 sfixed64.

# Any

Any 消息类型可以让你使用消息作为嵌入类型而不必持有他们的.proto定义. Any把任意序列化后的消息作为bytes包含, 带有一个URL, 工作起来类似一个全局唯一的标识符. 为了使用Any类型, 需要导入google/protobuf/any.proto.

```java
import "google/protobuf/any.proto";

message ErrorStatus {
  string message = 1;
  repeated Any details = 2;
}
```

给定消息类型的default type URL是 type.googleapis.com/packagename.messagename.

不同语言实现将提供运行时类库来帮助以类型安全的方式封包和解包Any的内容 - 例如, 在java中, Any类型将有特别的pack()和unpark()访问器, 而在c++中有PackFrom() 和 PackTo()方法:

```java
// 在Any中存储任务消息类型
NetworkErrorDetails details = ...;
ErrorStatus status;
status.add_details()->PackFrom(details);

// 从Any中读取任意消息
ErrorStatus status = ...;
for (const Any& detail : status.details()) {
  if (detail.IsType<NetworkErrorDetails>()) {
    NetworkErrorDetails network_error;
    detail.UnpackTo(&network_error);
    ... processing network_error ...
  }
}
```

**目前用于处理Any类型的运行时类库正在开发中**.

如果你熟悉proto2语法, Any 类型替代了extensions.

# Oneof

如果你有一个有很多字段的消息, 而同一时间最多只有一个字段会被设值, 你可以通过使用oneof特性来强化这个行为并节约内存.

Oneof 字段和常见字段类似, 除了所有字段共用内存, 并且同一时间最多有一个字段可以设值. 设值oneof的任何成员都将自动清除所有其他成员. 可以通过使用特殊的case()或者WhichOneof()方法来检查oneof中的哪个值被设值了(如果有), 取决于你选择的语言.

## 使用Oneof

使用oneof关键字来在.proto中定义oneof, 后面跟oneof名字, 在这个例子中是test_oneof:

```java
message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

然后再将oneof字段添加到oneof定义. 可以添加任意类型的字段, 但是不能使用重复(repeated)字段.

在生成的代码中, oneof字段和普通字段一样有同样的getter和setter方法. 也会有一个特别的方法用来检查哪个值(如果有)被设置了. 可以在所选语言的oneof API中找到更多信息.

## Oneof 特性

- 设置一个oneof字段会自动清除所有其他oneof成员. 所以如果设置多次oneof字段, 只有最后设置的字段依然有值.

    ```java
    SampleMessage message;
    message.set_name("name");
    CHECK(message.has_name());
    message.mutable_sub_message();   // Will clear name field.
    CHECK(!message.has_name());
    ```

- 如果解析器遇到同一个oneof的多个成员, 只有看到的最后一个成员在被解析的消息中被使用.
- oneof不能是重复字段
- Reflection APIs work for oneof fields. (反射api可以工作于oneof字段?)
- 如果使用c++, 确认代码不会导致内存奔溃. 下面的示例代码会导致crash因为sub_message已经在调用set_name()方法时被删除.

    ```java
    SampleMessage message;
    SubMessage* sub_message = message.mutable_sub_message();
    message.set_name("name");      // Will delete sub_message
    sub_message->set_...            // Crashes here
    ```

- 同样在c++中, 如果通过调用Swap()来交换两个带有oneof的消息, 每个消息将会有另外一个消息的oneof: 在下面这个案例中, msg1将会有sub_message和msg2会有name.

    ```java
    SampleMessage msg1;
    msg1.set_name("name");
    SampleMessage msg2;
    msg2.mutable_sub_message();
    msg1.swap(&msg2);
    CHECK(msg1.has_sub_message());
    CHECK(msg2.has_name());
    ```
## 向后兼容问题

当添加或者删除oneof字段时要小心. 如果检查oneof的值返回None/NOT_SET, 这意味着这个oneof没有被设置或者它被设置到一个字段, 而这个字段是在不同版本的oneof中. 没有办法区分这个差别, 因此没有办法知道一个未知字段是否是oneof的成员.


## 标签重用问题

- 移动字段进出oneof: 消息被序列化和解析后, 可能丢失部分信息(某些字段可能被清除).
- 删除oneof的一个字段加回来: 消息被序列化和解析后, 可能清除你当前设置的oneof字段
- 拆分或者合并oneof: 和移动普通字段一样有类似问题

# Maps

如果想常见一个关联的map作为数据定义的一部分, protocol buffers 提供方便的快捷语法:

```java
map<key_type, value_type> map_field = N;
```

key_type可以是任意整型或者字符类型(因此, 除了floating point和bytes外任何简单类型). value_type可以是任意类型.

因此, 例如, 如果你想创建一个projects的map, 每个Project消息都关联到一个string key, 可以这样定义:

```java
map<string, Project> projects = 3;
```

map字段不能重复. 另外注意map值的游历顺序是未定义的, 因此不能假设map元素保持特定顺序.

The generated map API is currently available for all proto3 supported languages. You can find out more about the map API for your chosen language in the relevant API reference.

所有proto3支持的语言的map API现在已经可用. 在相关的[API参考](https://developers.google.com/protocol-buffers/docs/reference/overview)可以找到更多关于所选语言的map API.

## 向后兼容

map语法和下面的代码等同, 因此不支持map的protocol buffers实现依然可以处理数据:

```java
message MapFieldEntry {
  key_type key = 1;
  value_type value = 2;
}

repeated MapFieldEntry map_field = N;
```

## 打包

可以在.proto文件中增加一个可选的包标记来防止protocol消息类型之间的名字冲突.

```java
package foo.bar;
message Open { ... }
```

在定义消息类型的字段时可以这样使用包标志:

```java
message Foo {
  ...
  foo.bar.Open open = 1;
  ...
}
```

包标志对生成代码的影响依赖所选的语言:

- 在c\++中, 生成类包裹在c\++ namespace中. 例如, Open将在namespace foo::bar中.
- 在java中, 包被作为java package使用, 除非你在.proto文件中显式提供java_package选项.
- 在Python中, 这个包指令将被忽略, 因为Python模块是根据他们在文件系统中的位置被组织的.
- 在Go中, 包被作为Go package使用, 除非你在.proto文件中显式提供go_package选项.
- 在Ruby中, 生成的代码被包裹在内嵌的Ruby namespaces, 转换为要求的Ruby capitalization风格(第一个字符大写;如果第一个字符不是字母则加一个PB_前缀). 例如, Open将在namespace Foo::Bar中.
- 在JavaNano中, 包被作为java package使用, 除非你在.proto文件中显式提供java_package选项.


## 包和命名解析

在protocol buffer语言中, 类型命名解析类似 C++: 首先搜索最内层的范围, 然后最内层的外一层, 以此类推, 每个包都被认为是它父包的"内层". 在最前面的'.'(例如, .foo.bar.Baz)意味着替换为从最外层范围开始.

protocol buffer编译器通过解析导入的.proto文件来解析所有的类型命名. 每个语言的代码生成器知道在这个语言中如何引用每个类型, 即使它有不同的范围规则.

# 定义服务

如果想在RPC (Remote Procedure Call) 系统中使用消息类型, 可以在.proto文件中定义RPC服务接口, 然后protocol buffer编译器会生成所选语言的服务接口代码和桩(stubs). 因此, 例如, 如果想定义一个RPC服务,带一个方法处理SearchRequest并返回SearchResponse, 可以在.proto文件中如下定义:

```java
service SearchService {
  rpc Search (SearchRequest) returns (SearchResponse);
}
```

使用protocol buffers最直接的RPC系统是gRPC: 一个Google开发的语言和平台无关的开源RPC系统. gRPC 可以非常好和protocol buffers一起工作并使用特别的protocol buffer编译器插件从.proto文件直接生成对应的RPC代码.

如果不想用gRPC, 也可以在自己的RPC实现中使用protocol buffers. 可以在[Proto2语言指南](https://developers.google.com/protocol-buffers/docs/proto#services)中找到更多这个消息.

也有一个第三方项目正在为Protocol Buffers开发RPC实现. 这里有一份我们知道的项目列表, 请见[third-party add-ons wiki page](https://github.com/google/protobuf/wiki/Third-Party-Add-ons).

## JSON 映射

Proto3支持JSON格式的标准编码, 让在系统之间分享数据变得容易. 编码在下面的表格中以type-by-type的基本原则进行描述.

如果一个值在json编码的数据中丢失或者它的值是null, 在被解析成protocol buffer时它将设置为对应的默认值.如果一个字段的值正好是protocol buffer的默认值, 这个字段默认就不会出现在json编码的数据中以便节约空间.具体实现应该提供选项来在json编码输出中出现带有默认值的字段.

注: 详细表格不翻译, 具体见[原文](https://developers.google.com/protocol-buffers/docs/proto3#json).

# 选项

.proto文件中的个别声明可以被一定数据的选项(option)注解. 选项不改变声明的整体意义, 但是在特定上下文会影响它被处理的方式. 可用选项的完整列表定义在google/protobuf/descriptor.proto.

有些选项是文件级别, 意味着他们应该写在顶级范围, 而不是在任何消息,枚举,或者服务定义之内. 有些选项时消息级别, 意味着他们应该写在消息定义内. 有些选项是字段级别, 意味着他们应该写在字段定义内. 选项也可以写在枚举类型, 枚举值, 服务定义和服务方法上. 但是, 目前没有任何有用的选项存在这些地方.

这里有一些最常用的选项:

- java_package (文件选项): 希望生成的java代码使用的package. 如果.proto文件中没有显式给出java_package选项, 则默认使用proto package(在.proto文件中通过"package"关键字指定). 然后, proto package通常不是一个好的java package因为protoc packages不会是期望的以反转的域名开始. 如果不生成java代码, 这个选项不会生效.

	> option java_package = "com.example.foo";

- java_outer_classname (文件选项): 想生成的最外面的java类名(同样也是文件名). 如果.proto文件没有显式指定java_outer_classname, 类名将通过把.proto文件的文件名转为为驼峰法(所以foo_bar.proto会得到FooBar.java)而产生.如果不生成java代码, 这个选项不会生效.

	> option java_outer_classname = "Ponycopter";

- optimize_for (文件选项): 可以设置为 SPEED(速度), CODE_SIZE(代码大小), or LITE_RUNTIME(轻运行时). 这会以下面的方式影响c++和java的代码生成器(也可能时第三方生成器):

	- SPEED (默认): protocol buffer编译器会为系列化, 解析和执行其他通用操作生成代码来处理消息类型. 代码会优化到极致.

	- CODE_SIZE: protocol buffer编译器会生成最少的类, 会依赖共享的, 基于反射的代码来实现序列化, 解析和其他操作. 生成的代码因此会被用SPEED生成的小很多, 但是操作更慢一些. 类会依然实现和在SPEED模式下完全相同的公开API. 这个模式主要用在包含非常多数量的.proto文件而又不需要他们都运行的极其快的应用中.

	- LITE_RUNTIME: protocol buffer编译器会生成仅仅依赖"轻"运行时类库(用libprotobuf-lite 替代 libprotobuf)的类. "轻"运行时类库比完全类库(around an order of magnitude smaller/注:看不懂这句话翻译不出...)小很多但是省略一些特性如描述符(?)和反射. 对于运行在受限制平台如移动手机上的应用特别有用. 编译器依然会生成所有方法的快速实现, 如同在SPEED模式下. 生成的类将仅仅实现每个语言的MessageLite接口, 仅仅提供完整Message接口方法的一个子集.

	> option optimize_for = CODE_SIZE;

- cc_enable_arenas (文件选项): 为生成c++代码开启 arena allocation.
- objc_class_prefix (文件选项): Sets the Objective-C class prefix which is prepended to all Objective-C generated classes and enums from this .proto. There is no default. You should use prefixes that are between 3-5 uppercase characters as recommended by Apple. Note that all 2 letter prefixes are reserved by Apple.
- packed (文件选项): 如果在重复字段或者基本数字类型上设置为true, 会使用一个更加紧凑的编码. 使用这个选项没有副作用. 但是, 注意在2.3.0版本之前, 接收到意外packed数据的解析器会忽略它. 因此, 将现有字段修改为packed格式是不可能不打破兼容性的. 在2.3.0和之后的版本中, 这个修改是安全的, 因为对于可压缩的字段解析器总是可以接收两个格式(压缩和不压缩), 但是在处理使用老版本protobuf的老程序请小心.

	> repeated int32 samples = 4 [packed=true];

- deprecated (字段选项): 如果设置为true, 表明这个字段被废弃, 新代码不应该再使用. 在大多数语言中这不会有实质影响. 在Java中, 这将会变成一个@Deprecated标签. 未来, 其他特定语言的代码生成器可能在字段的访问器上生成废弃标签, 在编译试图使用这个字段的代码时会生成警告. 如果这个字段不再被任何人使用而你想阻止新用户使用它, 可以考虑将字段声明替换为保留字段.

	> int32 old_field = 6 [deprecated=true];

## 自定义选项

Protocol Buffers也容许定义和使用自己的选项. 这是一个大多数人不需要的高级特性. 如果的确需要创建自己的选项, 请查看[Proto2 Language Guide]()来获取详情. 注意创建使用扩展的自定义选项, 是仅仅在proto3版本中被容许的.

# 生成你的类

为了生成Java, Python, C++, Go, Ruby, JavaNano, Objective-C, 或者 C# 代码, 需要处理定义在.proto文件中的消息类型, 需要在.proto文件上运行protocol buffer编译器protoc. 如果你没有安装这个编译器, 下载包并遵循README的指示. 对于Go, 需要为编译器安装特别的代码生成插件: 在github上的[golang/protobuf](https://github.com/golang/protobuf/)仓库中可以找到它和安装指示.

Protocol 编译器调用如下所示:

	protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --javanano_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto

- IMPORT_PATH 指定一个目录用于查找.proto文件, 当解析导入命令时. 缺省使用当前目录. 多个导入命令可以通过传递多次--proto_path选项来指定.这些路径将按照顺序被搜索. -I=IMPORT_PATH是--proto_path的缩写形式.
- 可以提供一个或者多个输出命令:

	* --java_out 生成 Java 代码在 DST_DIR. 详情见 [Java generated code reference](https://developers.google.com/protocol-buffers/docs/reference/java-generated).
	* 其他忽略请见原文

	作为一个额外的便利,如果 DST_DIR 以.zip 或者 .jar结束, 编译器会将输出写到一个给定名字的单独的zip格式的压缩文件中..jar 输出会有一个manifest文件如java jar规范要求的那样. 注意如果输出文件已经存在将被覆盖, 编译器不会添加文件到已经存在的文件中.

- 可以提供一个或多个.proto文件作为输入. 多个.proto文件可以一次指定. 虽然文件被以当前目录的相对路径命名, 每个文件必须位于一个IMPORT_PATH路径下, 以便编译器可以检测到它的标准名字.