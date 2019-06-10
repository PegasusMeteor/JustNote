# gRPC Concepts
<!-- TOC -->

- [gRPC Concepts](#grpc-concepts)
  - [Service definition](#service-definition)
  - [Using the API surface](#using-the-api-surface)
  - [Synchronous vs. asynchronous](#synchronous-vs-asynchronous)
- [RPC life cycle](#rpc-life-cycle)
  - [Unary RPC](#unary-rpc)
  - [Server streaming RPC](#server-streaming-rpc)
  - [Client streaming RPC](#client-streaming-rpc)
  - [Bidirectional streaming RPC](#bidirectional-streaming-rpc)
  - [Deadlines/Timeouts](#deadlinestimeouts)
  - [RPC termination](#rpc-termination)
  - [Cancelling RPCs](#cancelling-rpcs)
  - [Metadata](#metadata)
  - [Channels](#channels)

<!-- /TOC -->


接下来我们来介绍一下 gRPC中的一些核心概念。

## Service definition

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

## Using the API surface

从 `.proto`文件中的服务定义开始,gRPC 提供了能够生成server端和client端 代码的 `protocol buffer`编译插件。gRPC用户通常在客户端调用这些API，并在服务器端实现相应的API。

- 在服务端，服务端实现服务定义中声明的方法，并运行gRPC服务来处理客户端调用，gRPC基础结构解码request，执行服务方法并对response进行编码。
- 在客户端，客户端有一个称为存根的本地对象（对于某些语言，首选术语是客户端），它实现与服务相同的方法.然后客户端就可以在本地对象上调用这些方法，将调用的参数包装在适当的`protocol buffer`消息类型中——gRPC负责向服务器发送请求并返回服务器的`protocol buffer`响应.

## Synchronous vs. asynchronous

同步RPC调用在没有接收到服务端发送回来的response之前一直保持阻塞，这也是RPC调用中最期望的。 另一方面，网络本质上是异步的，在许多情况下，能够在不阻塞当前线程的情况下启动RPC非常有用。

# RPC life cycle

现在让我们仔细看看当gRPC客户端调用gRPC服务方法时会发生什么.

## Unary RPC

首先让我们看一下最简单的RPC类型，客户端发送单个请求并返回单个响应.

- 客户端在本地存根/客户端对象上调用方法之后，将会通知服务端已使用此调用的客户端元数据，方法名称和指定的截止时间（如果适用）调用RPC。
- 然后，服务端可以立即发送回自己的初始元数据（必须在任何响应之前发送），或者等待客户端的请求消息 --- 首先发生的是特定于应用程序的消息。
- 一旦服务端接受到了客户端的请求消息，它就会做一些相应的工作来构建response。然后服务端会将response返回给客户端，连同状态码以及其他的一些可选的状态信息一齐返回。
- 如果状态OK，客户端接收到了服务端的Response，从而完成客户端调用。

## Server streaming RPC

Server streaming RPC 类似于我们的简单示例，除了服务器在获取客户端的请求消息后发回响应流。服务端会将处理后的信息以及状态码和其他的可选状态信息一齐返回，客户端接收到此次响应，完成这次调用。

## Client streaming RPC

客户端流式RPC也类似于我们的简单示例，除了客户端向服务器发送请求流而不是单个请求。通常在收到了所有的客户端请求之后（但也不一定），服务端返回了包含状态信息以及其他可选信息的reponse，这次调用就结束了。

## Bidirectional streaming RPC

在双向流式RPC中，调用再次由调用方法的客户端和接收客户端元数据，方法名称和截止时间的服务端启动。服务端可以选择发回其初始元数据或等待客户端开始发送请求。

接下来会发生什么取决于应用程序，因为客户端和服务器可以按任何顺序读写 - 流完全独立地运行 。例如 服务端可以一直等待直到已经接收到了所有的客户端发送的消息之后才开始写入响应，或者服务端和客户端可以一直 `ping-pong`:服务器获取请求，然后发回响应，然后客户端根据响应发送另一个请求，依此类推。

## Deadlines/Timeouts

gRPC 允许客户端指定RPC调用完成的截止/超时时间，超时之后就会被 `DEADLINE_EXCEEDED` 错误中断。在服务端，服务端可以查询特定RPC是否已超时，或者剩余多少时间来完成RPC。

如果指定截止/超时时间因语言而异。有的语言api是在 截止到某一时间(固定时间点)，有的语言是在某一时间区间。

## RPC termination

在gRPC中，客户端和服务端都对这次调用是否成功进行独立的判断，它们的结论可能不匹配。这意味着，例如，可以在服务端成功完成RPC（“我已经发送了所有响应！”），但在客户端失败（“我的截止日期后响应才到达！”）。在客户端发送完所有请求之前，服务端也可以决定这次调用已经完成。

## Cancelling RPCs

客户端或服务端可以随时取消RPC.取消即立即终止RPC，以便不再进行进一步的工作。它不是“撤消”：取消之前所做的更改将不会被回滚。

## Metadata

元数据一般是键值对的形式，表示特定的RPC调用信息。key 一般是 strings，value一般也是string，有时可能是二进制数据。元数据对gRPC本身是不透明的 - 它允许客户端提供与服务器调用相关的信息，反之亦然。  

## Channels

gRPC通道提供与指定主机和端口上的gRPC服务器的连接，并在创建客户端存根（或某些语言中的“客户端”）时使用。客户端可以指定通道参数来修改gRPC的默认行为，例如打开和关闭消息压缩。通道具有状态，包括已连接和空闲。通道具有状态，包括已连接和空闲。

**以上就是gRPC的相关定义**
