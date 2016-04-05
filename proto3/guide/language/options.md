# 选项

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Options](https://developers.google.com/protocol-buffers/docs/proto3#options) 一节

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

Protocol Buffers也容许定义和使用自己的选项. 这是一个大多数人不需要的高级特性. 如果的确需要创建自己的选项, 请查看[Proto2 Language Guide](https://developers.google.com/protocol-buffers/docs/proto.html#customoptions)来获取详情. 注意创建自定义选项使用到[扩展](https://developers.google.com/protocol-buffers/docs/proto#extensions), 是仅仅在proto3版本中被容许的.

