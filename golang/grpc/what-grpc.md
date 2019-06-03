# What is gRPC

## OverView

在gRPC中，客户端应用程序可以直接调用不同计算机上的服务应用程序上的方法，就像它是本地对象一样，使我们可以更轻松地创建分布式应用程序和服务。 与许多RPC系统一样，gRPC基于定义服务的思想，指定可以使用其参数和返回类型远程调用的方法。 在服务器端,服务端实现此接口并运行gRPC服务来处理客户端调用。 在客户端，客户端有一个存根（在某些语言中称为客户端），它提供与服务器相同的方法。服务端与客户端一一对应。

![gRPC调用示意图](../images/landing-2.svg)

gRPC客户端和服务器可以在各种环境中相互运行和通信 ,并且可以使用任何gRPC支持的语言编写。 因此,我们可以使用Go，Python或Ruby轻松创建Java中的gRPC服务器。 此外，最新的Google API将具有gRPC版本的接口，我们可以轻松地在应用程序中构建Google功能。

## Working with Protocol Buffers

gRPC 默认采用 protocol buffers 数据传输格式。protocol buffers 是google开发的一种能够将结构数据序列化的数据描述语言。

使用protocol buffers的第一步是要在扩展名为.proto的proto文件中定义序列化的数据的结构。 Protocol buffer 数据会被结构化成一个 `message`,而这个 `message` 其实就是一条包含了一些属性（name-value对）的记录。例如下面的例子。

```protobuf
message Person {
  string name = 1;
  int32 id = 2;
  bool has_ponycopter = 3;
}
```

一旦我们指定好了数据结构，我们就可以使用`protocol buffer` 编译器`protoc`根据我们的 proto 定义 生成相应数据访问类了。编程语言可以是我们喜欢的任意语言。
这会为我们定义的结构中的成员对象提供简单的访问方法，以及将整个结构序列化和反序列化的一些方法。

例如在 [Go Quick Start](quick-start.md)例子中，我们定义的数据结构。  

```protobuf
// The greeting service definition.
service Greeter {
  // Sends a greeting
  rpc SayHello (HelloRequest) returns (HelloReply) {}

  // Sends another greeting
  rpc SayHelloAgain (HelloRequest) returns (HelloReply) {}
}

// The request message containing the user's name.
message HelloRequest {
  string name = 1;
}

// The response message containing the greetings
message HelloReply {
  string message = 1;
}
```

在经过 proto 编译生成之后，会有下面的一部分代码生成。从其中能够看到Getter方法以及一些序列化和反序列化的方法。

```go

// The request message containing the user's name.
type HelloRequest struct {
    Name                 string   `protobuf:"bytes,1,opt,name=name,proto3" json:"name,omitempty"`
    XXX_NoUnkeyedLiteral struct{} `json:"-"`
    XXX_unrecognized     []byte   `json:"-"`
    XXX_sizecache        int32    `json:"-"`
}

func (m *HelloRequest) Reset()         { *m = HelloRequest{} }
func (m *HelloRequest) String() string { return proto.CompactTextString(m) }
func (*HelloRequest) ProtoMessage()    {}
func (*HelloRequest) Descriptor() ([]byte, []int) {
    return fileDescriptor_17b8c58d586b62f2, []int{0}
}

func (m *HelloRequest) XXX_Unmarshal(b []byte) error {
    return xxx_messageInfo_HelloRequest.Unmarshal(m, b)
}
func (m *HelloRequest) XXX_Marshal(b []byte, deterministic bool) ([]byte, error) {
    return xxx_messageInfo_HelloRequest.Marshal(b, m, deterministic)
}
func (m *HelloRequest) XXX_Merge(src proto.Message) {
    xxx_messageInfo_HelloRequest.Merge(m, src)
}
func (m *HelloRequest) XXX_Size() int {
    return xxx_messageInfo_HelloRequest.Size(m)
}
func (m *HelloRequest) XXX_DiscardUnknown() {
    xxx_messageInfo_HelloRequest.DiscardUnknown(m)
}

var xxx_messageInfo_HelloRequest proto.InternalMessageInfo

func (m *HelloRequest) GetName() string {
    if m != nil {
        return m.Name
    }
    return ""
}

// The response message containing the greetings
type HelloReply struct {
    Message              string   `protobuf:"bytes,1,opt,name=message,proto3" json:"message,omitempty"`
    XXX_NoUnkeyedLiteral struct{} `json:"-"`
    XXX_unrecognized     []byte   `json:"-"`
    XXX_sizecache        int32    `json:"-"`
}

func (m *HelloReply) Reset()         { *m = HelloReply{} }
func (m *HelloReply) String() string { return proto.CompactTextString(m) }
func (*HelloReply) ProtoMessage()    {}
func (*HelloReply) Descriptor() ([]byte, []int) {
    return fileDescriptor_17b8c58d586b62f2, []int{1}
}

func (m *HelloReply) XXX_Unmarshal(b []byte) error {
    return xxx_messageInfo_HelloReply.Unmarshal(m, b)
}
func (m *HelloReply) XXX_Marshal(b []byte, deterministic bool) ([]byte, error) {
    return xxx_messageInfo_HelloReply.Marshal(b, m, deterministic)
}
func (m *HelloReply) XXX_Merge(src proto.Message) {
    xxx_messageInfo_HelloReply.Merge(m, src)
}
func (m *HelloReply) XXX_Size() int {
    return xxx_messageInfo_HelloReply.Size(m)
}
func (m *HelloReply) XXX_DiscardUnknown() {
    xxx_messageInfo_HelloReply.DiscardUnknown(m)
}

var xxx_messageInfo_HelloReply proto.InternalMessageInfo

func (m *HelloReply) GetMessage() string {
    if m != nil {
        return m.Message
    }
    return ""
}

```
