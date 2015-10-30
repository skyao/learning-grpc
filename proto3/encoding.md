编码
=======

注: 内容翻译来自官网资料 [Encoding](https://developers.google.com/protocol-buffers/docs/encoding).

这封文档描述protocol buffer消息的二进制格式. 在应用中使用protocol buffer不需要理解这些, 但是它对于了解不同的protocol buffer格式对编码消息的大小的影响非常有用.

# 简单消息

假设你有下面这个非常简单的消息定义:

```java
message Test1 {
  required int32 a = 1;
}
```

在应用中, 创建一个Test1消息并设置a为150. 然后序列化这个消息到输出流. 如果你可以检查编码后的消息, 你会看到3个字节:

	08 96 01

如此的小 - 但是这里发生了什么? 我们继续看.

# Base 128 Varints

为了理解上面简单的protocol buffer编码, 你首先需要理解varints. varints是使用一个或者多个字节序列化整型的方法. 越小的数字需要越少数量的字节.

在一个varint中的每个字节, 除了最后一个字节外, 有设置有most significant bit (msb) - The lower 7 bits of each byte are used to store the two's complement representation of the number in groups of 7 bits, least significant group first. (该怎么翻译?)

例如, 这里有一个数字1 - 这是一个字节, 因此msb不被设置:

	0000 0001

下面是300 - 这有一点复杂:

	1010 1100 0000 0010

怎么知道这是300呢? 首先将每个字节的msb去掉, 这个仅仅是告诉我们是否已经读到数字的结尾(你可以看到, 第一个字节被设置了,因为在varint中不止一个字节):

	1010 1100 0000 0010
	→ 010 1100  000 0010

把两个7bit的组翻转过来, 记得varints保存数字是least significant group first (汗). 然后可以将他们连接起来得到最后的值:

    000 0010  010 1100
    →  000 0010 ++ 010 1100
    →  100101100
    →  256 + 32 + 8 + 4 = 300

注: 为了更好的理解这个例子,我们从头到尾来推断一下300这个数字的编码过程

1. 整型300的标准32位(4字节)二进制表示为"00000000 00000000 00000001 00101100"
2. 从后向前每次按7bit分隔为"0000010 0101100", 剩下全是0的忽略
3. 翻转过来得到"0101100 0000010"
4. 为每个7bit增加msb, 前面7bit之前加1表示后面还有数据并凑成8bit为一个byte, 最后一个msb设置为0, 这样得到"10101100 00000010"

# 消息结构

你知道的, protocol buffer消息是一系列的键值对. 消息的二进制格式仅仅是用字段的数字作为key - 每个字段的名字和声明的类型仅仅在解码的最后通过引用消息类型定义(例如.proto文件)来检测.

When a message is encoded, the keys and values are concatenated into a byte stream. When the message is being decoded, the parser needs to be able to skip fields that it doesn't recognize. This way, new fields can be added to a message without breaking old programs that do not know about them. To this end, the "key" for each pair in a wire-format message is actually two values – the field number from your .proto file, plus a wire type that provides just enough information to find the length of the following value.

当消息被编码时, key和value被链接到一个字节流. 当消息被解码时, 解析器需要能跳过无法识别的字段. 这样, 新的字段可以被增加到消息而不打破旧的不知道新他们的程序. 在一个消息中每个键值对的"key"事实上有两个值 - 来从.proto文件的字段数字, 外加类型用于提供足够的信息来找到随后的值的长度.

可用的类型如下:

| Type | Meaning | Used For |
|--------|--------|--------|
|    0   | Varint | int32, int64, uint32, uint64, sint32, sint64, bool, enum |
|    1   | 64-bit | fixed64, sfixed64, double |
|    2   | Length-delimited | string, bytes, embedded messages, packed repeated fields |
|    3   | Start group | groups (deprecated) |
|    4   | End group | groups (deprecated) |
|    5   | 32-bit | fixed32, sfixed32, float |

在流化的消息中的每个key都是一个varint (field_number << 3) | wire_type - 换句话说, 数字的后三位保存着类型.

让我们再来看我们的简单例子. 现在你知道在流的第一个数字中总是一个varint key, 这里是08, 或则(去掉msb):

Now let's look at our simple example again. You now know that the first number in the stream is always a varint key, and here it's 08, or (dropping the msb):

	000 1000

You take the last three bits to get the wire type (0) and then right-shift by three to get the field number (1). So you now know that the tag is 1 and the following value is a varint. Using your varint-decoding knowledge from the previous section, you can see that the next two bytes store the value 150.

取后三位可以得到类型(0), 然后位移3位可以得到字段数字(1). 所以现在你知道标签是1, 后面的值是一个varint. 使用从上一章得到的varint解码知识, 可以知道后面2个字节存储的值是150.

    96 01 = 1001 0110  0000 0001
           → 000 0001  ++  001 0110 (去掉 msb 并掉转7 bits的组)
           → 10010110
           → 2 + 4 + 16 + 128 = 150

# 更多值类型

## 有符号整型

如你所见, 在上一节中, 所有和类型0关联的protocol buffer类型被编码为varints. 但是, 当编码负数的时候, 在有符号整型(sint32和sint64) 和 "标准" 整型类型(int32和int64)之间有一个重要的差别. 如果用int32或者int64作为一个负数的类型, 所得结果的varint总是10个字节长度 - 它被当成一个非常巨大的无符号整型处理. 如果使用有符号类型, 所得结果的varint使用更有效率的ZigZag编码.

ZigZag encoding maps signed integers to unsigned integers so that numbers with a small absolute value (for instance, -1) have a small varint encoded value too. It does this in a way that "zig-zags" back and forth through the positive and negative integers, so that -1 is encoded as 1, 1 is encoded as 2, -2 is encoded as 3, and so on, as you can see in the following table:

ZigZag 编码将有符号整型映射到无符号整型, 所有绝对值小的值(比如-1)数字会得到一个小的varint编码值. 实现的方式是"zig-zags", 在正数和负数整型之间来回摇摆, 因此-1被编码为1, 1被编码为2, -2被编码为3, 由此类推, 在下面的表格中可以看到:

| 原始有符号整型 | 编码结果 |
|--------|--------|
|    0   |    0   |
|   -1   |    1   |
|    1   |    2   |
|    2   |    3   |
| 2147483647  | 4294967294 |
| -2147483648 | 4294967295 |

换句话说, 对于sint32, 每个值 n 被编码为:

	(n << 1) ^ (n >> 31)

或者64位版本:

	(n << 1) ^ (n >> 63)

注意第二个移动 - (n >> 31)部分 - 是一个算数shift(arithmetic shift 怎么翻译好?). 因此, 用另外的话说, 移动的结果要么是0(如果n是正数) 要么是1(如果n是负数).

当sint32或者sint64被解析时, 它的值被解码回原始值, 有符号的版本.

## 非varint 数值

非varint 数值很简单 - double和fixed64是类型1, 解析器会期待一个固定64位的数据; 类似的float 和 fixed32是类型5, 解析器会期待一个32位. 这两个案例中值以little-endian/低字节序存储.

## 字符串

类型 2 (length-delimited) 意味着值是一个varint编码长度加指定数量的数据字节.

```java
message Test2 {
  required string b = 2;
}
```

设置b的值为"testing"会得到结果:

> 12 07 ++74 65 73 74 69 6e 67++

下划线字节是utf8编码的"testing". 这里的key是0x12 -> tag=2, type=2. varint的长度值是7, 我们发现后面跟随有7个字节 - 这是我们的字符串.


## 内嵌消息

这里是带有一个内嵌消息的消息定义, 内嵌消息是我们的范例类型 Test1:

```java
message Test3 {
  required Test1 c = 3;
}
```

And here's the encoded version, again with the Test1's a field set to 150:

这里是编码后的版本, Test1的字段再次设置为150:

	1a 03 08 96 01

As you can see, the last three bytes are exactly the same as our first example (08 96 01), and they're preceded by the number 3 – embedded messages are treated in exactly the same way as strings (wire type = 2).

可以看到, 最后三个字节和第一个例子里面完全相同(08 96 01), 并且他们前面有一个数字3 - 内嵌消息完全是和字符串(wire type = 2)一样对待.

注: 推导一下以便理解

1. 第一个字节1A的二进制是"0001 1010"
2. "0001 1010"的位移三位后结果是"011",表示字段的数字标签值是3, 对应消息定义里面的c=3
3. "0001 1010"的后三位"010"值是2, 表示wire type为2, Length-delimited
4. 从1A后按照varint读取长度, 03的结果是3, 表示后面有三个字节
5. 继续读取3个字节, 这是内嵌的消息Test1 c的内容, 然后按照Test1的定义继续解析这三个字节

# 可选和重复元素

如果消息定义中有重复元素(没有[packed=true]选项), 被编码的消息会有0个或者多次有相同标签数字的键值对. 这些重复值不需要连续出现; 他们可能夹杂着其他字段. 解析时元素必须相对的顺序会被保留, 但是和相对其他字段的顺序丢失了.

如果元素是可选的, 被编码的消息可能有也可能没有带有这个标签数字的键值对.

通常, 编码好的消息绝不会有一个可选或必需字段的多个实例. 但是, 解析器被期望能处理这种情况. 对于数值类型和字符串, 如果相同的值出现多次, 解析器接受它看到的最后一个值. 对于内嵌消息字段, 解析器合并同多个实例的同一个字段, 就像Message::MergeFrom方法一样 - 就是说, 后面的实例的所有scalar字段会覆盖前面的实例, scalar内嵌消息会合并, 而重复字段会连接. 这些规则的影响是: 解析连接在一起的两个编码后的消息和分别解析两个消息然后合并得到的对象, 结果是完全一样的. 也就是说, 这个方法:

```java
MyMessage message;
message.ParseFromString(str1 + str2);
```

等同于这个方式:

```java
MyMessage message, message2;
message.ParseFromString(str1);
message2.ParseFromString(str2);
message.MergeFrom(message2);
```

这个特性偶尔有用, 因为它容许你合并两个消息, 即使在你不知道他们类型的情况下.

## 打包重复字段

版本2.1.0引入了打包重复字段(packed repeated fields), 声明方式和重复字段类似但是带有一个[packed=true]选项. 工作方式类似重复字段, 但是编码不同. 不包含元素的打包重复字段不会出现在编码后的消息中. 另外, 字段的所有元素被打包到一个简单的键值对中, wire typt为2 (length-delimited). 每个元素和平常一样编码, 只是前面没有标签.

例如, 假设有消息类型:

```java
message Test4 {
  repeated int32 d = 4 [packed=true];
}
```

现在来构建一个Test4, 为重复字段d设置值3, 270 和 86942. 然后, 编码结果会是这样:

    22        // tag (field number 4, wire type 2)
    06        // payload size (6 bytes)
    03        // first element (varint 3)
    8E 02     // second element (varint 270)
    9E A7 05  // third element (varint 86942)

只有原生数字类型(类型为varint, 32位或者64位wire类型)的重复字段才可以声明为"packed".

注意, 虽然通常没有理由为一个打包重复字段编码多个键值对, 编码器必须准备接受多个键值对. 在这种情况下, 负载将被连接合并. 每个键值对必须包含完整数量的元素(Each pair must contain a whole number of elements, 这里有点不懂既然负载都合并了,也就只剩下一个键值对可, 何来each pair?).

# 字段顺序

无论你在.proto文件中以任何顺序使用字段数字, 当消息被序列化时, 它已知的字段应该按照字段数字顺序写入, 在提供的c++,java和python序列化代码中是如此. 这容许解析代码使用基于字段数字顺序的优化. 当然, protocolbuffer解析器必须能够解析任何顺序的字段, 毕竟不是所有消息都是被简单从对象系列化得来的 - 例如, 有时通过简单的连接来合并两个消息.

如果一个消息有未知字段, 当前java和c++实现会在顺序写入已知字段之后按照任意顺序写入他们. 当前Python实现不跟踪(trace)未知字段.
