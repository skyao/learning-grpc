Channel层
=============

`Channel` 提供在 `transport` 层之上的抽象，设计用来被 `stub` 调用。Channel 和相关的 `ClientCall` 和 `ClientCall.Listener` 交换被解析的请求(request)和应答(response)对象，而 transport 层仅仅处理序列化后的数据。

应用可以通过使用 `ClientInterceptor` 装饰 `Channel` 来实现添加通用的横切行为到 stub 上。预期大部分应用代码将不会直接使用这个类，而是使用已经绑定到 Channel 的 stub， 而 Channel 在应用初始化时已经被装饰好。

> 注： 上面两段内容翻译自 类 io.grpc.Channel 的 javadoc.
