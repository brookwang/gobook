cmux是一个通用的Go库，用于复用连接。使用cmux，您可以在同一TCP侦听器上提供gRPC，SSH，HTTPS，HTTP，Go RPC以及几乎任何其他协议。
### 地址
https://github.com/soheilhy/cmux

### 示例
`
// Create the main listener.
l, err := net.Listen("tcp", ":23456")
if err != nil {
	log.Fatal(err)
}

// Create a cmux.
m := cmux.New(l)

// Match connections in order:
// First grpc, then HTTP, and otherwise Go RPC/TCP.
grpcL := m.Match(cmux.HTTP2HeaderField("content-type", "application/grpc"))
httpL := m.Match(cmux.HTTP1Fast())
trpcL := m.Match(cmux.Any()) // Any means anything that is not yet matched.

// Create your protocol servers.
grpcS := grpc.NewServer()
grpchello.RegisterGreeterServer(grpcS, &server{})

httpS := &http.Server{
	Handler: &helloHTTP1Handler{},
}

trpcS := rpc.NewServer()
trpcS.Register(&ExampleRPCRcvr{})

// Use the muxed listeners for your servers.
go grpcS.Serve(grpcL)
go httpS.Serve(httpL)
go trpcS.Accept(trpcL)

// Start serving!
m.Serve()`


### 性能
由于只匹配连接的第一个字节，因此长连接（即RPC和流水线HTTP流）的性能开销可以忽略不计。

### 其他
* TLS：net/http使用类型断言来识别TLS连接; 由于cmux的lookahead-implementation连接包装了底层的TLS连接，因此这种类型的断言失败了。
因此，您可以使用cmux提供HTTPS，但http.Request.TLS 不会在处理程序中设置。
* 同一连接上的不同协议：cmux在接受时匹配连接。例如，一个连接可以是gRPC或REST，但不能同时使用两者。也就是说，我们假设客户端连接用于gRPC或REST。
* Java gRPC客户端：Java gRPC客户端阻塞，直到它从服务器接收到SETTINGS帧。如果您使用Java客户端连接到cmux的gRPC服务器，请与编写者匹配：
