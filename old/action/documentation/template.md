# 使用模板定制输出

## 模板使用方式

protoc-gen-doc 插件支持模板，可以通过使用不同的模板来定制输出的内容和格式，命令如下：

```bash
protoc --doc_out=/usr/local/include/dolphin/api.mustache,index.html:../../../target/contract-doc userService.proto
```

只是简单的将原来 `--doc_out=html,*` 中的 `html` 修改为具体的模板路径。

## 定制模板

默认使用的模板在 `protoc-gen-doc` 源代码根目录下的 `templates` 文件夹中：

```bash
$ ls
docbook.mustache  html.mustache  markdown.mustache  scalar_value_types.json
```

定制时，将 `html.mustache` 文件复制出来再修改就是，例如:

```bash
sudo cp html.mustache /usr/local/include/dolphin/api.mustache
```

## HTML模板结构

TBD