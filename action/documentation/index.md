文档生成
===========

# protoc-gen-doc

https://github.com/estan/protoc-gen-doc

> 这是一个Google Protocol Buffers编译器(protoc)的文档生成插件。这个插件可以从.proto文件中的注释内容生成HTML, DocBook 或者 Markdown 文档。

## 安装

参考 [protoc-gen-doc](https://github.com/estan/protoc-gen-doc) Installation章节的信息。

### linux安装

对于ubuntu系统，参考 [protoc-gen-doc](https://software.opensuse.org/download.html?project=home%3Aestan%3Aprotoc-gen-doc&package=protoc-gen-doc)的说明。

对于 xUbuntu 14.04，请运行以下命令：

```bash
sudo sh -c "echo 'deb http://download.opensuse.org/repositories/home:/estan:/protoc-gen-doc/xUbuntu_14.04/ /' >> /etc/apt/sources.list.d/protoc-gen-doc.list"
sudo apt-get update
sudo apt-get install protoc-gen-doc
```

### windows

protoc-gen-doc的github项目的 [release page上](https://github.com/estan/protoc-gen-doc/releases) ，可以找到windows的版本的下载。

## 执行

```bash
cd dolphin-demo/contract
protoc --doc_out=html,index.html:target/contract-doc src/main/proto/*.proto
```

可惜，报错：

	src/main/proto/user.proto:1:10: Unrecognized syntax identifier "proto3".  This parser only recognizes "proto2".

目前版本还不支持proto3！真是遗憾。

## 后记

跑去看了一下，作者正在开发1.0版本，承诺会加入proto3的支持，只能期待这个版本早点发布了。

- [Add support for protobuf 3.0](https://github.com/estan/protoc-gen-doc/issues/17): github上的issue记录

