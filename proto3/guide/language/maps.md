# Maps

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Maps](https://developers.google.com/protocol-buffers/docs/proto3#maps) 一节

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

