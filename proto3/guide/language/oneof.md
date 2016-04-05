# Oneof

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Oneof](https://developers.google.com/protocol-buffers/docs/proto3#oneof) 一节

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


