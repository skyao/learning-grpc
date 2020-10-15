# build 插件

原有插件生成的 HTML 文件内容和格式并不理想，考虑自行调整。

因此 fork 了原有仓库，准备动手修改。

这样就有必要能自己从c的源代码开始编译打包。

参考原有的插件打包说明：

https://github.com/skyao/protoc-gen-doc/blob/master/BUILDING.md

## 准备工作

按照要求，需要准备两个东西：

- Protocol Buffers library from Google
- QtCore from Qt 5

先执行命令：

```bash
sudo apt-get install qt5-qmake qt5-default libprotobuf-dev protobuf-compiler libprotoc-dev
```

另外g++肯定是必备的了，没有安装的话先安装：

```bash
sudo apt-get install g++
```

### make

在插件的代码根目录下，执行下面命令：

```bash
qmake
make
```

此时如果发现报错，无法include <google/protobuf/stubs/common.h> ：

    /usr/include/google/protobuf/compiler/plugin.h:58:42: fatal error: google/protobuf/stubs/common.h: No such file or directory
     #include <google/protobuf/stubs/common.h>
                                              ^
    compilation terminated.
    make: *** [main.o] Error 1

说明第一个前提条件 "Protocol Buffers library from Google" 没有满足，需要先安装。

### 安装Protocol Buffers library

参考 protocol buffer 的 `C++ Installation - Unix`：

https://github.com/google/protobuf/blob/master/src/README.md#c-installation---unix

先安装各种工具：

```bash
sudo apt-get install autoconf automake libtool curl make g++ unzip
```

然后准备开始build。注意这里有两个方式，对应着protocol release页面有两个下载地址：

- protobuf-cpp-3.0.0.zip
- Source code (tar.gz)

1. 从source code开始：下载 `Source code (tar.gz)` ，然后开始build，因为这个包中没有configure脚本文件，因此需要执行 `./autogen.sh` 来生成。而这个过程中需要下载jmock来做测试，但是jmock下载时又遇到无法下载的问题。所以不建议用这个方式

2. 从release package中，也就是下载 `protobuf-cpp-3.0.0.zip`，这里面有现成的configure脚本文件，一次执行下面的命令：

    ```bash
    ./configure
    make
    make check
    sudo make install
    sudo ldconfig
    ```

make完成之后，上面缺失的头文件就可以在下列地址找到了：

	/usr/local/include/google/protobuf/stubs/common.h

### 清理

用上面的方式安装 protocol buffer 之后，protoc被安装在新的位置了：

```bash
$which protoc
/usr/local/bin/protoc
```

此时可以删除掉之前安装的版本：

```bash
sudo rm -rf /usr/bin/protoc
```

### 继续 make plugin

继续执行 proto-gen-doc 的make，这次就可以成功了：

```bash
$ make
g++ -m64 -Wl,-O1 -o protoc-gen-doc mustache.o main.o qrc_protoc-gen-doc.o   -lprotoc -pthread -L/usr/local/lib -lprotobuf -lQt5Core -lpthread
```

完成之后，在当前目录下就会得到生成的 `protoc-gen-doc` 文件。

这个文件不需要安装，直接复制到 `/usr/local/bin/protoc-gen-doc` 就好了, 另外顺便清理一下之前安装在  `/usr/bin/protoc-gen-doc` 的版本：

```bash
sudo cp protoc-gen-doc /usr/local/bin/
sudo rm /usr/bin/protoc-gen-doc
```

使用时发现，之前放置在 `/usr/include/` 下的include文件现在又找不到了，需要同样移动到 `/usr/local/include/` 下：

```bash
$ protoc --doc_out=html,index.html:../../../target/contract-doc userService.proto
dolphin/dolphinDescriptor.proto: File not found.
userService.proto: Import "dolphin/dolphinDescriptor.proto" was not found or had errors.

# 移过去就好了
sudo mv /usr/include/dolphin/ /usr/local/include/
```

## 在CentOS6上安装

在centos 6 上安装时，遇到更多问题，记录下来备用。

1. /usr/local/bin 值得PATH路径问题

上面的方法build并安装 protobuf-cpp 之后，protoc被成功安装在 `/usr/local/bin/protoc`，直接执行 protoc 也可以找到，但是用jenkins就是报错找不到 protoc 。

只好在jenkins的shell脚本里面加入一句：

```bash
export PATH=/usr/local/bin:$PATH
```

随后protoc执行时报错说找不到 libprotoc.so.10 ，但是这个文件是有build到 `/usr/local/lib` 的，只要再加一句，将 `/usr/local/lib` 加入到LD_LIBRARY_PATH：

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

然后继续报错，这回是 GLIBC 版本太久：

```bash
protoc-gen-doc: /lib64/libc.so.6: version `GLIBC_2.14' not found (required by protoc-gen-doc)
protoc-gen-doc: /usr/lib64/libstdc++.so.6: version `GLIBCXX_3.4.14' not found (required by protoc-gen-doc)
--doc_out: protoc-gen-doc: Plugin failed with status code 1.
```

安装 GLIBC 2.14 版本(参考 http://blog.csdn.net/dodo_check/article/details/9341145)：

```bash
wget http://mirror.bjtu.edu.cn/gnu/libc/glibc-2.14.tar.xz
xz -d glibc-2.14.tar.xz
tar xvf glibc-2.14.tar
cd glibc-2.14
mkdir build
cd build
./configure
make
make install
```

不幸报错：

```bash
/root/glibc-2.14/build/elf/ldconfig: Can't open configuration file /usr/local/lib/glic/etc/ld.so.conf: No such file or directory
make[1]: Leaving directory `/root/glibc-2.14'
```



