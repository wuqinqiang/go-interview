---
weight: 3
bookFlatSection: true
title: "初识 protobuf"
---

# 初始protobuf

### Protobuf 是什么

`protobuf`全称是`protocal buffers` ，平时我们都会简称为`pb`，下文就使用`pb`来介绍了。

官网中对pb的完整定义如下:

> Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages.

简单的说， `pb`是一种与语言、平台无关，可扩展的序列化数据格式。它的定位类似于`JSON`、`XML`，但是比他们更小、更快、更简单。

对于开发者来说，我们只需要定义`proto`文件。然后使用对应的`IDL`编译器把`.proto文件`编译成对应语言的代码即可。

从某个角度来看，`.proto`文件就是接口。里面包含了接口方法、入参、出参。



### 环境

#### pb

下载安装`pb`  [项目地址](https://github.com/protocolbuffers/protobuf)。

如果是`linux`使用`apt`或者`apt-get`

```shell
apt install -y protobuf-compiler
protoc --version  # 确认版本为 3+
```

如果是Mac`,使用`Homebrew`,

```shell
brew install protobuf
protoc --version # 确认版本为 3+
```

官方建议使用`proto3`教程中使用的`proto3` ，如果想了解 `proto3`相比`proto2`的改变可以查看这里[[Protocol Buffers v3.0.0](https://github.com/protocolbuffers/protobuf/releases/tag/v3.0.0)](https://github.com/protocolbuffers/protobuf/releases/tag/v3.0.0)。



#### Go plugins

`protobuf`核心的工具是`C++`开发的。在官方的编译器中并不支持`Go`语言(好奇的是为啥其他主语言都支持，是个谜)。

所以，如果我们想要在` Go`中使用`protobuf` ，还需要安装能支持在`Go`中编译`protobuf`插件，这样我们就可以 `.proto`文件生成对应的`Go`代码了。

执行以下命令安装插件最新版本：

```shell
go get -u github.com/golang/protobuf/protoc-gen-go
```

你也可以指定版本号。

```shell
$ go get google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
```

当然了项目中使用的 `grpc`,我们需要安装对应 `protobuf`编译生成`grpc`代码的插件。

```shell
go get -u google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1 
```

如果有需要，更新一下`PATH`，让`protoc`命令找到插件。

```shell
export PATH="$PATH:$(go env GOPATH)/bin"
```

顺便提一嘴，可以看到这里我使用了`go install`，那么`go get`和`go install`有什么区别?

{{< hint info >}}
go install=go build +把 go build 编译后生成的可执行文件放到 GOPATH/bin 目录下。 {{< /hint >}}



{{< hint warning >}}
go get = git clone + go install。

 {{< /hint >}}





### 如何使用

我们先来看一个简单的示例，我们创建一个`order.proto`文件，内容如下:

```go
syntax = "proto3";

option go_package = ".;order";

message Order {
  string order_id = 1;
  string user_id = 2;
  uint32 order_amount = 3;
  bool is_cancel = 4;
}
```

逐行解释。

`pb`有两个版本，默认版本是`proto2`，如果要使用`proto3`，我们就得在**非空非注释**的第一行代码中使用`syntax = "proto3";` 声明版本。 

`option`表示可选项配置。有些选项是文件级别的，需要在一个文件的顶级作用域定义，比如上面的`option go_package = "order";`。

`go_package`选项，定义的值，就是把`pb`文件编译生成`Go`代码后文件的`package`名。这样其他生成的`Go`代码就可以通过关键字`import`使用到这个包。

你可能还会在`.proto`文件中看到类似 `package xxx`这样的， 比如，

```
syntax = "proto3";
package order;
option go_package = ".;order";
```

`package`主要用来解决不同的`pb`文件存在相同消息类型名称之间的冲突。比如`a.proto`有一个`Common message`，`b.proto` 也有一个 `Common message`，此时需要通过`package`来做区分。 

那`package`和`go_package` 有什么关系吗？

> There is no correlation between the Go import path and the [`package` specifier](https://developers.google.com/protocol-buffers/docs/proto3#packages) in the `.proto` file. The latter is only relevant to the protobuf namespace, while the former is only relevant to the Go namespace. Also, there is no correlation between the Go import path and the `.proto` import path.

**没有关系**。

`package`只和`pb`空间有关，而 `go_package` 前面解释过了，和`Go`的`import`包有关，他们两者，本质上不是一个层面的。

接着我们定义了一个`Order`的消息`message`类型，定义消息类型是使用关键字`message` 定义的。

`Order`定义了四个字段，对应`string`、`uint32`以及`bool`字段类型。

消息类型中字段定义的格式为，

```protobuf
[ "singular|repeated｜......" ] type fieldName "=" fieldNumber [ "[" fieldOptions "]" ] ";"
```

 默认情况下，每个字段最前面的修饰符为`singular`，一般省略不写。

假设一个主订单下有多个子订单，我们可以如何定义？

```
syntax = "proto3";

option go_package = ".;order";

message Order {
  string order_id = 1;
  string user_id = 2;
  uint32 order_amount = 3;
  bool is_cancel = 4;
  repeated OrderItem items = 5;
}

message OrderItem {
  string title = 1;
  string image = 2;
}
```

我们新定义了一个`OrderItem`子订单的消息类型。然后在`Order`中，我们使用了`OrderItem`作为`items`字段的类型值。

`repeated`是另一种修饰符，表示允许字段重复。上面的场景即一个订单有多个子订单的概念。

编译成`Go`代码，`items`会变成`slice`类型。

其他类型就不一一介绍，更多的类型可以查看这：[message type](https://developers.google.com/protocol-buffers/docs/proto3?hl=en#simple)



接着我们通过命令编译(确保工具已装)生成Go相关的代码，执行如下命令：

```shell
protoc --proto_path=. --go_out=. *.proto
```

`--proto_path`指定要编译的源码的搜索路径，上面的`.`表示当前目录。如果不指定`--proto_path`的话,默认为`pwd`。所以上面我们也可以省略成这样，

```shell
protoc --go_out=. *.proto
```

`--go_out` 参数告知`protoc`编译器去加载对应的`protoc-gen-go`工具且指定编译生成`Go`代码后保存位置。当然如果是生成`php`代码那就是`--php_out`。对应的语言，对应`--xx_out`。

最后的`*.proto`表示搜索当前目录下所有`.proto`文件。

那么整句命令行意思就是：编译当前目录下所有`.proto`文件， 把生成的`Go`代码存放到当前目录。

上面的命令执行后，当前目录下多了一个`order.pb.go`文件。生成的数据结构:

```go

// 主订单
type Order struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	OrderId     string       `protobuf:"bytes,1,opt,name=order_id,json=orderId,proto3" json:"order_id,omitempty"`
	UserId      string       `protobuf:"bytes,2,opt,name=user_id,json=userId,proto3" json:"user_id,omitempty"`
	OrderAmount uint32       `protobuf:"varint,4,opt,name=order_amount,json=orderAmount,proto3" json:"order_amount,omitempty"`
	IsCancel    bool         `protobuf:"varint,5,opt,name=is_cancel,json=isCancel,proto3" json:"is_cancel,omitempty"`
	Items       []*OrderItem `protobuf:"bytes,6,rep,name=items,proto3" json:"items,omitempty"`
}

........

func (x *Order) GetOrderId() string {
	if x != nil {
		return x.OrderId
	}
	return ""
}

func (x *Order) GetUserId() string {
	if x != nil {
		return x.UserId
	}
	return ""
}

func (x *Order) GetOrderAmount() uint32 {
	if x != nil {
		return x.OrderAmount
	}
	return 0
}
......

// 子订单
type OrderItem struct {
	state         protoimpl.MessageState
	sizeCache     protoimpl.SizeCache
	unknownFields protoimpl.UnknownFields

	Title string `protobuf:"bytes,1,opt,name=title,proto3" json:"title,omitempty"`
	Image string `protobuf:"bytes,2,opt,name=image,proto3" json:"image,omitempty"`
}
......
```

可以看到生成的主订单结构中，`Items` 是一个切片类型。同时生成的还有一些`Getxxx`方法，方便读取字段值。

等等，接口呢？

`message` 有了，现在我们可以在`.proto`文件定义`RPC`服务接口了。

```go
syntax = "proto3";

option go_package = ".;order";

service OrderService {
  rpc GetOrderList (OrderListReq) returns (OrderListResp) {}
}

message OrderListReq {
  string user_id = 1;
}

message OrderListResp {
  repeated Order list = 1;
}

message Order {
  string order_id = 1;
  string user_id = 2;
  uint32 order_amount = 3;
  bool is_cancel = 4;
  repeated OrderItem items = 5;
}

message OrderItem {
  string title = 1;
  string image = 2;
}
```

我们定义了一个 `OrderService` 的服务，提供了一个`GetOrderList`的接口，它的入参是`OrderListReq`，返回类型是`OrderListResp`，是不是很像我们平常定义的函数。

和平常自定义函数的不同在于，可以自定义无入参出参的函数，而在`pb`中接口必须携带参数，否则编译的时候会报，

```go
Expected type name
```

针对不需要参数的情况，我们一般会定义一个空`message`类型或者传入`google.protobuf.Empty`，传入`google.protobuf.Empty` 不好点在于不易扩展，万一这个接口要参数了呢。

`service`定义完毕，再次执行上述命令，你会发现，重新生成的`order.pb.go`文件并没有变化。这是因为`RPC`框架很多，`protoc`编译器并不知道给我们生成哪个`RPC`框架的代码。

`protoc-gen-go`内部已经集成了一个名为`gRPC`的插件，所以我们可以通过命令行参数`--go-grpc_out`生成`gRPC`代码：

```shell
protoc --go_out=.  --go-grpc_out=. *.proto
```

然后你就发现这个错误，

![](https://cdn.syst.top/2021-08-28.png)

我们需要改动 `option go_package = ".;order"` 为 `option go_package = "./;order"`。

再次执行，多出了一个`order_grpc.pb.go`文件。里面就能看到`grpc`客户端和服务端的`api`服务。

```go
// OrderServiceClient is the client API for OrderService service.
type OrderServiceClient interface {
	GetOrderList(ctx context.Context, in *OrderListReq, opts ...grpc.CallOption) (*OrderListResp, error)
}

func NewOrderServiceClient(cc grpc.ClientConnInterface) OrderServiceClient {
	return &orderServiceClient{cc}
}

func (c *orderServiceClient) GetOrderList(ctx context.Context, in *OrderListReq, opts ...grpc.CallOption) (*OrderListResp, error) {
	out := new(OrderListResp)
	err := c.cc.Invoke(ctx, "/OrderService/GetOrderList", in, out, opts...)
	if err != nil {
		return nil, err
	}
	return out, nil
}

......

// OrderServiceServer is the server API for OrderService service.
type OrderServiceServer interface {
	GetOrderList(context.Context, *OrderListReq) (*OrderListResp, error)
	mustEmbedUnimplementedOrderServiceServer()
}

func RegisterOrderServiceServer(s grpc.ServiceRegistrar, srv OrderServiceServer) {
	s.RegisterService(&OrderService_ServiceDesc, srv)
}
```



### 总结

这篇主要介绍了`pb`一些基础的知识，包括定义`message`、`service`以及通过`protoc-gen-go`生成对应的`Go`代码，`gRPC`代码。关于 `protoc`命令，后面有必要再单独写一篇文章。

下一篇，通过生成的`Go`代码，完成`gRPC`通信，然后正式步入主题。



### 参考

- https://grpc.io/docs/languages/go/quickstart/
- https://developers.google.com/protocol-buffers
- https://colobu.com/2019/10/03/protobuf-ultimate-tutorial-in-go/#option
- https://chai2010.cn/advanced-go-programming-book/ch4-rpc/ch4-02-pb-intro.html
