# gRPC Concepts
<!-- TOC -->

- [gRPC Concepts](#grpc-concepts)
  - [服务定义](#%E6%9C%8D%E5%8A%A1%E5%AE%9A%E4%B9%89)

<!-- /TOC -->


接下来我们来介绍一下 gRPC中的一些核心概念。

## 服务定义

与许多RPC系统一样，gRPC基于定义服务的思想，指定可以使用其参数和返回类型的远程调用的方法. 默认情况下，gRPC使用 `protocol buffers` 作为接口定义语言（IDL）来描述服务接口和消息结构。  如果需要，可以使用其他替代方案.

```protobuf
service HelloService {
  rpc SayHello (HelloRequest) returns (HelloResponse);
}

message HelloRequest {
  string greeting = 1;
}

message HelloResponse {
  string reply = 1;
}
```

gRPC 允许定义四种服务方法:

- 一元RPC(Unary RPCs)，客户端向服务器发送单个请求并返回单个响应，就像正常的函数调用一样。

```protobuf
rpc SayHello(HelloRequest) returns (HelloResponse){
}
```

- 服务端流式RPC（Server streaming RPCs ），客户端向服务器发送请求并获取流以读取消息序列。客户端从返回的流中读取，直到没有更多消息。 gRPC保证单个RPC调用中的消息排序。

```protobuf
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse){
}
```

- 客户端流式RPC(Client streaming RPCs)，客户端再次使用提供的流写入一系列消息并将其发送到服务器.一旦客户端写完消息，它就等待服务器读取它们并返回它的响应。 gRPC再次保证在单个RPC调用中的消息排序。

```protobuf
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse) {
}
```

- 双向流式RPC(Bidirectional streaming RPCs)，双方使用读写流发送一系列消息。这两个流独立运行，因此客户端和服务器可以按照自己喜欢的顺序进行读写.例如，服务端可以在写入其响应之前等待接收所有客户端消息，或者它可以交替地读取消息然后写入消息，或者读取和写入的其他组合。两个流中的消息的顺序不会发生变化。

```protobuf
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse){
}
```
我们将在下面的RPC生命周期部分中更详细地介绍不同类型的RPC.

