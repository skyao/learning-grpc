# 支持proto3

## 安装 protoc-gen-doc

简单遵循安装要求即可：

https://github.com/estan/protoc-gen-doc#installation

安装完成之后的protoc是2.5.0版本，无法处理proto3的文件。因此我们需要升级替换protoc为v3.0.0版本。

## 升级protoc

### 使用预编译版本

1. 下载

    请先在 protobuf 的 [发布页面](https://github.com/google/protobuf/releases) 中找到对应版本的 download ，然后下载对应版本的 protoc, 如 v3.0.0的 linux 64位版本：

    - [protoc-3.0.0-linux-x86_64.zip](https://github.com/google/protobuf/releases/download/v3.0.0/protoc-3.0.0-linux-x86_64.zip)

2. 清理

	如果之前有安装过 protoc 的老版本，如2.5.0版本，则需要在安装前删除旧有版本的protoc和include文件，操作如下：

    ```bash
	$ protoc --version
    libprotoc 2.5.0
    # 上面老版本是 2.5.0，需要删除
    which protoc
	/usr/bin/protoc
    # 删除 protoc
    sudo rm /usr/bin/protoc
    # 删除默认的include文件
    sudo rm -rf /usr/include/google/protobuf/
    ```

3. 安装

	解压缩得到的zip文件，然后执行下面的操作，复制protoc和include文件：

    ```bash
    mkdir temp
    mv protoc-3.0.0-linux-x86_64.zip temp
    cd temp
	unzip protoc-3.0.0-linux-x86_64.zip
    ```

    输出如下(直接解压缩到当前目录，所以解压前最好准备一个临时目录，如上面shell命令中的temp目录):

        Archive:  protoc-3.0.0-linux-x86_64.zip
        creating: include/
        creating: include/google/
        creating: include/google/protobuf/
        inflating: include/google/protobuf/struct.proto
        inflating: include/google/protobuf/type.proto
        inflating: include/google/protobuf/descriptor.proto
        inflating: include/google/protobuf/api.proto
        inflating: include/google/protobuf/empty.proto
        creating: include/google/protobuf/compiler/
        inflating: include/google/protobuf/compiler/plugin.proto
        inflating: include/google/protobuf/any.proto
        inflating: include/google/protobuf/field_mask.proto
        inflating: include/google/protobuf/wrappers.proto
        inflating: include/google/protobuf/timestamp.proto
        inflating: include/google/protobuf/duration.proto
        inflating: include/google/protobuf/source_context.proto
        creating: bin/
        inflating: bin/protoc
        inflating: readme.txt

	开始复制文件并修改文件权限为755：

    ```bash
    # 复制protoc文件
    sudo cp bin/protoc /usr/bin/
    sudo chmod 755 /usr/bin/protoc
    # 复制include文件
    sudo cp -r include/google/protobuf/ /usr/include/google/
    sudo chmod -R 755 /usr/include/google/protobuf
    ```

## 生成文档

参考 protoc-gen-doc 的使用说明：

https://github.com/estan/protoc-gen-doc#invoking-the-plugin

注意：存放生成文件的目录必须预先建立，否则会报错。

```bash
// 我们的proto文件在这里
cd contract/src/main/proto
// 准备生成HTML文件，并放置在target/contract-doc目录下
mkdir ../../../target/contract-doc
protoc --doc_out=html,index.html:../../../target/contract-doc userService.proto
```

## 解决 import 问题

执行 protoc 时，如果 userService.proto 还 import 了 `google/protobuf` 之外的其他的 .proto 文件，则会因为无法找到需要的 import 文件而报错。

有三种方式解决：

1. 直接将要import的proto文件复制到当前目录，保证路径正确
2. 将需要import的proto文件复制到 `/usr/include/` 目录

    ```bash
    sudo cp -r dolphin/ /usr/include/
    sudo chmod -R 755 /usr/include/dolphin/
    ```

3. 使用 `--proto_path=PATH` 在 protoc 的命令行参数中添加import的路径，而且这个选项可以使用多次

## 总结

现在至少可以用了，可以得到生成的HTML文件，虽然还有很多不足，后续再更新。

## 后续工作

后面就可以考虑：

1. 在jenkins上建立上述环境
2. 然后添加job来为各个项目的contract文件生成HTML文档
3. 再将生成的文档统一发布到nginx之类的服务器下

这样就可以实现一个简单的文档中心，用于各个服务API文档的统一维护和发布。



