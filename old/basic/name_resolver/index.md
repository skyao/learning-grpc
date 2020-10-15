# NameResolver

NameResolver 是可拔插的组件，用于解析目标 URI 并返回地址给调用者。

NameResolver 使用 URI 的 scheme 来检测是否可以解析它， 再使用 scheme 后面的组件来做实际处理。

目标的地址和属性可能随着时间的过去发生修改，因此调用者注册 Listener 来接收持续更新。






