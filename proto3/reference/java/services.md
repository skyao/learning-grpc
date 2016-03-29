服务
=========

> 注：内容翻译自官网参考文档中 Java Generated Code 的 [Services](https://developers.google.com/protocol-buffers/docs/reference/java-generated#service) 一节。

如果.proto文件中包含下列行：

	option java_generic_services = true;

那么protocol buffer编译器将如在这节中描述的那样生成基于文件中的服务定义的代码。不过，生成的代码可能不适合，因为它没有绑定到任何特定的RPC系统，而且为此需要更多让代码适应一个系统的间接层次。如果不想生成这些代码，在文件中增加这行：

	option java_generic_services = false;

> 注：这是protocol buffer的做法，和grpc无关。grpc中不需要设置这个选项。

如果上述两行没有给出，选项默认为false，通用服务将被弃用。(注意从2.4.0开始，这个选项默认为true)

基于.proto语言服务定义的RPC系统应该提供 插件 来生成适用于系统的代码。这些插件大多要求禁用抽象服务(abstract service)，以便他们可以生成他们自己的同样名字的类。插件是版本2.3.0新出现的(2010年一月)

这节的剩余部分描述当abstract services开启时protocol buffer会生成什么。

- Interface
- Stub
- Blocking Interfaces

> 注： 这里就不详细翻译了，因为我主要关注grpc的使用，在没有grpc的情况下如何生成通用的service对我来说意义不大。

