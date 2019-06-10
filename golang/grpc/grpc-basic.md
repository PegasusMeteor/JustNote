# gRPC Basics - Go

<!-- TOC -->

- [gRPC Basics - Go](#grpc-basics---go)
  - [Example Code](#example-code)
  - [定义服务](#%E5%AE%9A%E4%B9%89%E6%9C%8D%E5%8A%A1)
  - [生成server端和client 端代码](#%E7%94%9F%E6%88%90server%E7%AB%AF%E5%92%8Cclient-%E7%AB%AF%E4%BB%A3%E7%A0%81)
  - [编写Server端](#%E7%BC%96%E5%86%99server%E7%AB%AF)
    - [实现接口](#%E5%AE%9E%E7%8E%B0%E6%8E%A5%E5%8F%A3)

<!-- /TOC -->

接下来我们就开始以go为基础去使用gRPC。本文将介绍：

- 在 `.proto` 文件中进行服务定义。
- 使用 `protocol buffer` 编译器生成 server 端 和client 端代码。
- 使用 Go gRPC API 为我们定义的服务编写简单的 client和server。

## Example Code

我们这里使用的示例是  [grpc/grpc-go/examples/route_guide](https://github.com/grpc/grpc-go/tree/master/examples/route_guide), 为了使用这个示例，可以先下载 `grpc-go` 这个库。

```shell
go get google.golang.org/grpc
```

然后切换到 `grpc/grpc-go/examples/route_guide`目录下。

```shell
cd $GOPATH/src/google.golang.org/grpc/examples/route_guide
```

## 定义服务

从前面的介绍中我们已经介绍如何使用 `protocol buffers`来定义服务以及返回值类型。

```protocol
// Interface exported by the server.
service RouteGuide {
  // 一元RPC
  // A simple RPC.
  //
  // Obtains the feature at a given position.
  //
  // A feature with an empty name is returned if there's no feature at the given
  // position.
  rpc GetFeature(Point) returns (Feature) {}

  // 服务端流式RPC
  // A server-to-client streaming RPC.
  //
  // Obtains the Features available within the given Rectangle.  Results are
  // streamed rather than returned at once (e.g. in a response message with a
  // repeated field), as the rectangle may cover a large area and contain a
  // huge number of features.
  rpc ListFeatures(Rectangle) returns (stream Feature) {}

  // 客户端流式RPC
  // A client-to-server streaming RPC.
  //
  // Accepts a stream of Points on a route being traversed, returning a
  // RouteSummary when traversal is completed.
  rpc RecordRoute(stream Point) returns (RouteSummary) {}

  // 双向流式RPC
  // A Bidirectional streaming RPC.
  //
  // Accepts a stream of RouteNotes sent while a route is being traversed,
  // while receiving other RouteNotes (e.g. from other users).
  rpc RouteChat(stream RouteNote) returns (stream RouteNote) {}
}
```

同时在 `.proto` 文件中还定义了用于请求和响应的message 类型。

```proto
// Points are represented as latitude-longitude pairs in the E7 representation
// (degrees multiplied by 10**7 and rounded to the nearest integer).
// Latitudes should be in the range +/- 90 degrees and longitude should be in
// the range +/- 180 degrees (inclusive).
message Point {
  int32 latitude = 1;
  int32 longitude = 2;
}

// A latitude-longitude rectangle, represented as two diagonally opposite
// points "lo" and "hi".
message Rectangle {
  // One corner of the rectangle.
  Point lo = 1;

  // The other corner of the rectangle.
  Point hi = 2;
}

// A feature names something at a given point.
//
// If a feature could not be named, the name is empty.
message Feature {
  // The name of the feature.
  string name = 1;

  // The point where the feature is detected.
  Point location = 2;
}

// A RouteNote is a message sent while at a given point.
message RouteNote {
  // The location from which the message is sent.
  Point location = 1;

  // The message to be sent.
  string message = 2;
}

// A RouteSummary is received in response to a RecordRoute rpc.
//
// It contains the number of individual points received, the number of
// detected features, and the total distance covered as the cumulative sum of
// the distance between each point.
message RouteSummary {
  // The number of points received.
  int32 point_count = 1;

  // The number of known features passed while traversing the route.
  int32 feature_count = 2;

  // The distance covered in metres.
  int32 distance = 3;

  // The duration of the traversal in seconds.
  int32 elapsed_time = 4;
}


```

## 生成server端和client 端代码

对 `route_guide`目录执行以下的命令。将会在该目录下 生成一个route_guide.pb.go 的go 文件。

```shell
protoc -I routeguide/ routeguide/route_guide.proto --go_out=plugins=grpc:routeguide
```

这里面将包含：

- 所有的 `protocol buffers`代码用于填充，序列化和检索我们的请求和响应消息类型。
- 客户端使用RouteGuide服务中定义的方法时，调用的接口类型。
- 服务端要实现的接口类型，以及RouteGuide服务中定义的方法

## 编写Server端

gRPC已经为我们自动生成了编写Server端需要实现的接口和方法。接下来我们就来实现一个gRPC Server。一共有两个过程：

- 实现服务定义中的接口
- 运行gRPC服务,以侦听来自客户端的请求并将其分派给正确的服务实现。

具体示例可以在  [grpc-go/examples/route_guide/server/server.go](https://github.com/grpc/grpc-go/blob/master/examples/route_guide/server/server.go)中查看。接下来我们分析下，具体都干了些什么？

### 实现接口

可以看到有一个 `routeGuideServer`的结构体，实现了 `RouteGuideServer` 接口

```go
type routeGuideServer struct {
        ...
}
...

func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
        ...
}
...

func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
        ...
}
...

func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
        ...
}
...

func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
        ...
}
...
```

`routeGuideServer` 实现了所有的服务接口.

**Simple RPC**

我们先以最简单的(Simple RPC)为例，来看看他是具体是怎么实现的。他是从客户端获取一个point，并返回一个Feature。

```go

// GetFeature returns the feature at the given point.
func (s *routeGuideServer) GetFeature(ctx context.Context, point *pb.Point) (*pb.Feature, error) {
    for _, feature := range s.savedFeatures {
        if proto.Equal(feature.Location, point) {
            return feature, nil
        }
    }
    // No feature was found, return an unnamed feature
    return &pb.Feature{Location: point}, nil
}
```

该方法传递了 RPC 上下文对象和客户端的 `Point` `protocol buffers` 请求。它返回一个Feature协议缓冲区对象，其中包含响应信息和 error。我们使用适当的信息填充Feature，然后将其与nil错误一起返回，告诉gRPC我们已经完成了RPC的处理，并且Feature可以返回给客户端。

**Server-side streaming RPC**

接下来看下服务端流式RPC。ListFeatures是服务器端流式RPC，因为我们需要发送多个 Features 给客户端。

```go
// ListFeatures lists all features contained within the given bounding Rectangle.
func (s *routeGuideServer) ListFeatures(rect *pb.Rectangle, stream pb.RouteGuide_ListFeaturesServer) error {
    for _, feature := range s.savedFeatures {
        if inRange(feature.Location, rect) {
            if err := stream.Send(feature); err != nil {
                return err
            }
        }
    }
    return nil
}

```

如你所见，这次与简单请求的request和reponse不同，这里获取了两个比较复杂的参数`pb.Rectangle`, `RouteGuide_ListFeaturesServer`来进行处理并构建response。在这个方法里，我们构建了多个 Feature 对象，并使用send方法将他们写入到`RouteGuide_ListFeaturesServer`中。返回了一个nil的错误。gRPC层会将其转换为适当的RPC状态，以便在链路中发送。

**Client-side streaming RPC**

https://grpc.io/docs/tutorials/basic/go/