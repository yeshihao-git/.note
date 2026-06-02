---
tags:
  - cpp
---
# protobuf

**what**：
（Protocol Buffers）google开发的 语言无关 、 平台无关 的 结构化数据序列化协议 ，体积更小、解析更快、但可读性差（相比 JSON）；定义在 .proto文件，使用 protoc 编译生成 .pb.h、.pb.cc 文件

## protoc

**what**：
编译器，用于将符合 Protocol Buffers 协议的 .proto文件 编译成 服务端、客户端代码（不同语言的转换工作由对应的语言插件实现）

### protoc生成cpp代码

- 生成cpp代码：--cpp_out=./out：指定输出目录 ./out，会生成 example.pb.h 和 example.pb.cc
```bash
protoc --cpp_out=./out example.proto
```
- 生成 gRPC 代码：--grpc_out 生成 example.grpc.pb.h 和 example.grpc.pb.cc
```bash
protoc --grpc_out=./out --plugin=protoc-gen-grpc=`which grpc_cpp_plugin` example.proto
```

