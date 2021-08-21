---
weight: 1
bookFlatSection: true
title: "环境安装"
---

# 开发环境



#### Go环境

首先是 `Go` ，直接安装最新的版本就好。这个很简单，官网地址:[go安装](https://golang.org/doc/install)。



#### Protocol buffer

`protocol buffer` ，简称 `protobuf`。是一种与语言、平台无关 、可扩展的序列化结构数据格式。官方有一句话:

{{< hint info >}}
 Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler.{{< /hint>}}

所以它的定位类似于 `xml` 、`json`，但是从性能上和效率上大幅度优于 `xml`、`json`。

但是它也有个缺点，可读性差，毕竟是以二进制方式存储的。



下载安装 `protocol buffer`  [项目地址](https://github.com/protocolbuffers/protobuf)。

如果是 `linux` 使用 `apt` 或者 `apt-get`

```shell
apt install -y protobuf-compiler
protoc --version  # 确认版本为 3+
```

如果是 `Mac`,使用 `Homebrew`,

```shell
brew install protobuf
protoc --version # 确认版本为 3+
```

官方建议使用 `proto3` 教程中使用的 `proto3` ，如果想了解  `proto3` 相比 `proto2` 的改变可以查看这里[[Protocol Buffers v3.0.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.0.0)](https://github.com/protocolbuffers/protobuf/releases/tag/v3.0.0)。



#### Go plugins

`protobuf` 核心的工具是 `C++` 开发的。在官方的编译器中并不支持 `Go` 语言(好奇的是为啥其他主语言都支持，是个谜)。

所以，如果我们想要在` Go` 中使用 `protobuf` ，还需要安装能支持在 `Go` 中 编译 `protobuf` 插件，这样我们就可以把 `.proto` 文件生成对应的 `Go` 代码了。

执行以下命令安装插件最新版本：

```shell
go get -u github.com/golang/protobuf/protoc-gen-go
```

当然你也可以指定版本号。

```shell
$ go get google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
```

当然了项目中使用的 `grpc`,我们需要安装对应 `protobuf` 编译生成 `grpc` 代码 的插件。

```shell
go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1 
```

如果有需要，更新一下 `PATH`，让 `protoc` 命令找到插件。

```shell
export PATH="$PATH:$(go env GOPATH)/bin"
```

顺便提一嘴，可以看到这里我使用了 `go install`，那么 `go get` 和 `go install` 有什么区别?

{{< hint info >}}
go install=go build +把 go build 编译后生成的可执行文件放到 GOPATH/bin 目录下。 {{< /hint >}}



{{< hint warning >}}
go get = git clone + go install。

 {{< /hint >}}





很清晰了吧。

