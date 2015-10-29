风格指南
=======

注: 内容翻译来自官网资料 [Style Guide](https://developers.google.com/protocol-buffers/docs/style).

这个文档为.proto文件提供风格指南. 通过遵循下列约定, 可以让protocol buffer消息定义和他们对应的类保持一致并容易阅读.

# 消息和字段名

消息名使用驼峰法 - 例如, SongServerRequest. 字段名私用下划线分隔名 - 例如, song_name.

```java
message SongServerRequest {
  required string song_name = 1;
}
```

为字段名使用这种命名约定可以得到如下的访问器:


C++:

```java
  const string& song_name() { ... }
  void set_song_name(const string& x) { ... }
```

Java:

```java
  public String getSongName() { ... }
  public Builder setSongName(String v) { ... }
```

# 枚举

枚举类型名使用驼峰法(首字母大写), 值的名字使用大写加下划线分隔:

```java
enum Foo {
  FIRST_VALUE = 1;
  SECOND_VALUE = 2;
}
```

每个枚举值以分号(;)结束, 不要用逗号(,).

# 服务

如果.proto文件定义RPC服务, 服务名和任何rpc方法应该用驼峰法(首字母大写):

```java
service FooService {
  rpc GetSomething(FooRequest) returns (FooResponse);
}
```
