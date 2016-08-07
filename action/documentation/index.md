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

> 2016-08-07 更新： 发现上面的错误提示只是 protoc 版本的问题，只要升级 protoc 版本到v3.0.0，这个 proto-gen-doc 插件依然可以正常工作！ 详细做法请见下一节 "支持proto3".



