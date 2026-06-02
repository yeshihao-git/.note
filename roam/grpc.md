---
tags:
  - cpp
---
# grpc

**what**：
google 开源的高性能 RPC（远程过程调用）框架 ，使得对服务器上方法的调用就像在本地一样（开发者只需关注业务逻辑无需关注底层网络通信），用于构建分布式系统，支持多种编程语言
  1. 基于 HTTP/2.0 和 Protocol Buffers

**how**：使用流程 
1. .proto文件中定义：message格式、服务、服务内rpc方法
```cpp
// 4 种 rpc 类型
// 一元rpc方法  ：客户端发送一个请求，服务端返回一个响应
rpc SayHello(HelloRequest) returns (HelloResponse);
// 服务器流式RPC：客户端发送一个请求，服务端返回一个流（一系列消息）
rpc LotsOfReplies(HelloRequest) returns (stream HelloResponse);
// 客户端流式RPC：客户端发送一个流，服务器返回一个响应
rpc LotsOfGreetings(stream HelloRequest) returns (HelloResponse);
// 双向流式RPC  ：双发收发流
rpc BidiHello(stream HelloRequest) returns (stream HelloResponse);
```
2. protoc、grpc插件：生成.pb、.grpc.pb文件
   - .pb.h和.pb.cc：message的序列化和反序列化代码
   - .grpc.pb.h和.grpc.pb.cc：服务和存根的框架代码
2. grpc客户端：生成 Channel，在 Channel 基础上生成 Stub（存根），通过 Stub 调用 grpc服务端方法
3. grpc服务端：
   - 服务实现类 继承 grpc.pb 文件中的 Service类，重写所有rpc方法
```cpp
class ChatServiceImpl final: public ChatService::Service { /*...*/ }
```
   - 启动服务：ServerBuilder工厂类 配置监听端口 并 注册服务实现类，启动
```cpp
chatserver::ChatServiceImpl service;
grpc::ServerBuilder builder;
builder.AddListeningPort(server_address, grpc::InsecureServerCredentials());
builder.RegisterService(&service);
std::unique_ptr<grpc::Server> server(builder.BuildAndStart());
std::thread  grpc_server_thread([&server]() {
	server->Wait();
});
```

## 编译grpc
1. git走代理
2. git clone -b <选择Tag> https://github.com/grpc/grpc
3. cd grpc
4. git submodule update --init
5. mkdir -p cmake/build
6. cd cmake/build
7. cmake -DCMAKE_CXX_STANDARD=17 ../..
8. make -j$(nproc)
9. make install
10. 安装到 /usr/local/bin，有些程序在 /usr/bin下寻找 -> 解决方式：软连接