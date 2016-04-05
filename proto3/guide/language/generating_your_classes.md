# 生成类

> 注：内容翻译自官网文档 Language Guide (proto3) 中的 [Generating Your Classes](https://developers.google.com/protocol-buffers/docs/proto3#generating) 一节

为了生成Java, Python, C++, Go, Ruby, JavaNano, Objective-C, 或者 C# 代码, 需要处理定义在.proto文件中的消息类型, 需要在.proto文件上运行protocol buffer编译器protoc. 如果你没有安装这个编译器, 下载包并遵循README的指示. 对于Go, 需要为编译器安装特别的代码生成插件: 在github上的[golang/protobuf](https://github.com/golang/protobuf/)仓库中可以找到它和安装指示.

Protocol 编译器调用如下所示:

	protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --javanano_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto

- IMPORT_PATH 指定一个目录用于查找.proto文件, 当解析导入命令时. 缺省使用当前目录. 多个导入命令可以通过传递多次--proto_path选项来指定.这些路径将按照顺序被搜索. -I=IMPORT_PATH是--proto_path的缩写形式.
- 可以提供一个或者多个输出命令:

	* --java_out 生成 Java 代码在 DST_DIR. 详情见 [Java generated code reference](https://developers.google.com/protocol-buffers/docs/reference/java-generated).
	* 其他忽略请见原文

	作为一个额外的便利,如果 DST_DIR 以.zip 或者 .jar结束, 编译器会将输出写到一个给定名字的单独的zip格式的压缩文件中..jar 输出会有一个manifest文件如java jar规范要求的那样. 注意如果输出文件已经存在将被覆盖, 编译器不会添加文件到已经存在的文件中.

- 可以提供一个或多个.proto文件作为输入. 多个.proto文件可以一次指定. 虽然文件被以当前目录的相对路径命名, 每个文件必须位于一个IMPORT_PATH路径下, 以便编译器可以检测到它的标准名字.