# gRPC Basics - Go

<!-- TOC -->

- [gRPC Basics - Go](#grpc-basics---go)
  - [Example Code](#example-code)
  - [定义服务](#%E5%AE%9A%E4%B9%89%E6%9C%8D%E5%8A%A1)
  - [生成server端和client 端代码](#%E7%94%9F%E6%88%90server%E7%AB%AF%E5%92%8Cclient-%E7%AB%AF%E4%BB%A3%E7%A0%81)
  - [编写Server端](#%E7%BC%96%E5%86%99server%E7%AB%AF)
    - [实现接口](#%E5%AE%9E%E7%8E%B0%E6%8E%A5%E5%8F%A3)
    - [启动server端](#%E5%90%AF%E5%8A%A8server%E7%AB%AF)
  - [编写client端](#%E7%BC%96%E5%86%99client%E7%AB%AF)
    - [创建一个存根/客户端](#%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%AD%98%E6%A0%B9%E5%AE%A2%E6%88%B7%E7%AB%AF)
    - [调用服务端方法](#%E8%B0%83%E7%94%A8%E6%9C%8D%E5%8A%A1%E7%AB%AF%E6%96%B9%E6%B3%95)

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

接下来来看一个比较复杂的。客户端流式gRPC.服务端从客户端获取一系列的`Points`返回单个`RouteSummary`.

```go
// RecordRoute records a route composited of a sequence of points.
//
// It gets a stream of points, and responds with statistics about the "trip":
// number of points,  number of known features visited, total distance traveled, and
// total time spent.
func (s *routeGuideServer) RecordRoute(stream pb.RouteGuide_RecordRouteServer) error {
    var pointCount, featureCount, distance int32
    var lastPoint *pb.Point
    startTime := time.Now()
    for {
        point, err := stream.Recv()
        if err == io.EOF {
            endTime := time.Now()
            return stream.SendAndClose(&pb.RouteSummary{
                PointCount:   pointCount,
                FeatureCount: featureCount,
                Distance:     distance,
                ElapsedTime:  int32(endTime.Sub(startTime).Seconds()),
            })
        }
        if err != nil {
            return err
        }
        pointCount++
        for _, feature := range s.savedFeatures {
            if proto.Equal(feature.Location, point) {
                featureCount++
            }
        }
        if lastPoint != nil {
            distance += calcDistance(lastPoint, point)
        }
        lastPoint = point
    }
}

```

正如我们所看到的，这次没有接收任何的request 请求参数，而是直接接收了一个 `RouteGuide_RecordRouteServer`流。服务端通过这个流**读写**消息。读的话，使用`Recv()`方法，返回单个response 使用它的 `SendAndClose()` 方法。

在这个方法体中，我们使用 `RouteGuide_RecordRouteServer`的`Recv()` 方法，重复读取客户端的请求到某一个实体中（本例中是point）,直到读取不出任何消息为止。 服务端需要去检查每一次Read返回的错误。如果error为nil，说明当前流没有问题，可以继续读取。如果是`io.EOF`，说明当前读取已经结束，server端可以返回 `RouteSummary`,如果它有其他类型的值，则将其原样返回。RPC会自动将其转换成相应的RPC 状态。

**Bidirectional streaming RPC**

废话不多说，上来先看下代码

```go

// RouteChat receives a stream of message/location pairs, and responds with a stream of all
// previous messages at each of those locations.
func (s *routeGuideServer) RouteChat(stream pb.RouteGuide_RouteChatServer) error {
    for {
        in, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }
        key := serialize(in.Location)

        s.mu.Lock()
        s.routeNotes[key] = append(s.routeNotes[key], in)
        // Note: this copy prevents blocking other clients while serving this one.
        // We don't need to do a deep copy, because elements in the slice are
        // insert-only and never modified.
        rn := make([]*pb.RouteNote, len(s.routeNotes[key]))
        copy(rn, s.routeNotes[key])
        s.mu.Unlock()

        for _, note := range rn {
            if err := stream.Send(note); err != nil {
                return err
            }
        }
    }
}
```

这次我们又接收到了`RouteGuide_RouteChatServer`流，与客户端流式RPC一样，仍然可以使用它来读写数据。但是，这次我们可以在客户端还在向stream中写数据的同时就返回值。

从代码中可以看出，server端这里读写数据的方法非常类似于客户端流式RPC里面的方法，无非就是这里使用了 `send()`方法，而不是`SendAndClose()`方法。因为send可以发送多个reponse。尽管每一端都能够按照对方发送消息的顺序读取其相应的消息，但是客户端和服务端都可以按照任何顺序进行读写，这两个流完全独立运行。

### 启动server端

一旦实现了所有的方法，我们就需要开启一个gRPC server，让客户端能够访问我们的服务。

```go
flag.Parse()
    lis, err := net.Listen("tcp", fmt.Sprintf("localhost:%d", *port))
    if err != nil {
        log.Fatalf("failed to listen: %v", err)
    }
    // 是否使用TLS
    var opts []grpc.ServerOption
    if *tls {
        if *certFile == "" {
            *certFile = testdata.Path("server1.pem")
        }
        if *keyFile == "" {
            *keyFile = testdata.Path("server1.key")
        }
        creds, err := credentials.NewServerTLSFromFile(*certFile, *keyFile)
        if err != nil {
            log.Fatalf("Failed to generate credentials %v", err)
        }
        opts = []grpc.ServerOption{grpc.Creds(creds)}
    }
    grpcServer := grpc.NewServer(opts...)
    pb.RegisterRouteGuideServer(grpcServer, newServer())
    grpcServer.Serve(lis)
```

- 指定端口
- 启动 gRPC server实例
- 将我们实现的service注册到 the gRPC server.
- `Serve()`方法启动，`Stop()`方法停止

## 编写client端

接下来就开始编写一个 gRPC client 来访问 `RouteGuide` 服务。 示例 代码 [grpc-go/examples/route_guide/client/client.go](https://github.com/grpc/grpc-go/blob/master/examples/route_guide/client/client.go)  

### 创建一个存根/客户端

为了调用服务端发方法，需要创建一个 gRPC channel 来跟server端进行通信。使用 `grpc.Dial()`方法，传入server端的地址和端口来实现。

```go
conn, err := grpc.Dial(*serverAddr)
if err != nil {
    ...
}
defer conn.Close()
```

还可以使用 `DialOptions` 在 `grpc.Dial` 设置使用证书认证等，例如下面这样

```go
flag.Parse()
    var opts []grpc.DialOption
    if *tls {
        if *caFile == "" {
            *caFile = testdata.Path("ca.pem")
        }
        creds, err := credentials.NewClientTLSFromFile(*caFile, *serverHostOverride)
        if err != nil {
            log.Fatalf("Failed to create TLS credentials %v", err)
        }
        opts = append(opts, grpc.WithTransportCredentials(creds))
    } else {
        opts = append(opts, grpc.WithInsecure())
    }
    conn, err := grpc.Dial(*serverAddr, opts...)
    if err != nil {
        log.Fatalf("fail to dial: %v", err)
    }
    defer conn.Close()
```

gRPC channel 建立好之后，需要一个client stub来 执行RPC调用。我们使用 `.proto` 文件中生成的 `pb` 包中的 `NewRouteGuideClient` 方法来创建。

```go
client := pb.NewRouteGuideClient(conn)

```

### 调用服务端方法

接下来我们看客户端如何调用服务端方法。**请注意,在gRPC-Go中，RPC以阻塞/同步模式运行，这意味着RPC调用等待服务器响应，并将返回响应或错误。**

**Simple RPC**

```go
  feature, err := client.GetFeature(context.Background(), &pb.Point{409146138, -746188906})
  if err != nil {
        ...
  }
```

就像调用本地方法一样简单，在方法中我们构造了一个 `protocol buffers` 对象 `db.Point`,我们还传递了一个context.Context对象，它允许我们在必要时更改RPC的行为，例如rpc调用还在进行时取消RPC. 如果没有出现错误，我们就能收到 server端的返回值。

看一下完整的写法。

```go
// printFeature gets the feature for the given point.
func printFeature(client pb.RouteGuideClient, point *pb.Point) {
   log.Printf("Getting feature for point (%d, %d)", point.Latitude, point.Longitude)
   ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
   defer cancel()
   feature, err := client.GetFeature(ctx, point)
   if err != nil {
      log.Fatalf("%v.GetFeatures(_) = _, %v: ", client, err)
   }
   log.Println(feature)
}

```

**Server-side streaming RPC**

接下来调用服务端流式RPC方法 `ListFeatures`，这个方法返回一系列的地理数据 `Features`.

```go
rect := &pb.Rectangle{ ... }  // initialize a pb.Rectangle
stream, err := client.ListFeatures(context.Background(), rect)
if err != nil {
    ...
}
for {
    feature, err := stream.Recv()
    if err == io.EOF {
        break
    }
    if err != nil {
        log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
    }
    log.Println(feature)
}
```

与简单的RPC调用一样，传入了context.Context来进行request，但是与其不一样的是，这次没有简单收到了一个repsonse对象，而是接收到了一个`RouteGuide_ListFeaturesClient`实例。客户端会使用 `RouteGuide_ListFeaturesClient` 来读取server端的消息。

我们使用 `RouteGuide_ListFeaturesClient`的`Recv（）`方法重复读取服务端对  `protocol buffers`对象（在本例中为Feature）的响应，直到没有更多消息为止。客户端需要对每一个调用中返回的error进行判断，错误的处理方式与server端类似。

看下完整的写法。

```go
// printFeatures lists all the features within the given bounding Rectangle.
func printFeatures(client pb.RouteGuideClient, rect *pb.Rectangle) {
   log.Printf("Looking for features within %v", rect)
   ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
   defer cancel()
   stream, err := client.ListFeatures(ctx, rect)
   if err != nil {
      log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
   }
   for {
      feature, err := stream.Recv()
      if err == io.EOF {
         break
      }
      if err != nil {
         log.Fatalf("%v.ListFeatures(_) = _, %v", client, err)
      }
      log.Println(feature)
   }
}
```

**Client-side streaming RPC**

未完待续

https://grpc.io/docs/tutorials/basic/go/