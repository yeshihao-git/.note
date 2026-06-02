---
tags:
  - cpp
  - boost
---
# asio

**what**：
跨平台的 C++异步IO库，通过异步操作实现高效的网络通信（默认 proactor模式 ，同时支持 reactor模式 ）
1. windows下：默认基于 IOCP 实现 proactor模式
2. linux下：能通过 epoll 实现 reactor 模式，也能通过扩展实现 proactor模式

核心组件 ：
1. io_context：IO执行上下文，用于连接 IO对象 和 操作系统的IO服务
2. IO对象：socket、定时器
3. 异步操作：async_read、async_write、async_accept等
4. completion handler：完成处理函数
```cpp
socket.async_read_some(buffer, [](error_code e, size_t) { /*处理逻辑*/ } );
// 此示例的完成处理函数是lambda
```
5. strand：保证回调函数在多线程环境下序列化进行

```cpp
/* 这些组件协作的示例 */
// 初始上下文对象
asio::io_context io_context;

// 创建IO对象，关联上下文对象
asio::ip::tcp::socket socket(io_context);
socket.connect(asio::ip::tcp::endpoint(asio::ip::address::from_string("127.0.0.1"), 8080));

// 使用IO对象，发起异步操作，并且注册回调函数（异步操作结束后调用）
char buffer[1024];
asio::async_read(socket, asio::buffer(buffer),
  [&](std::error_code ec, std::size_t length) {
    if (!ec) {
      // 处理读取的数据
      std::cout << "Received: " << std::string(buffer, length) << std::endl;
    }
  });

// 启动事件循环
io_context.run();  // 阻塞，直到所有操作完成
```

异步操作原理 ：
创建 io_context 对象，将 io_context 与 IO对象关联（eg：socket），进行异步操作，异步操作 中注册 completion handler，发起异步操作后立刻返回，后续同步阻塞（阻塞是等待操作系统IO完成通知，而不是主动轮询检查IO状态）的调用 boost::asio::io_context::run() ，内部会运行事件循环，并处理事件循环中任务队列里的任务，比如异步操作中注册的 completion handler（除此之外还有 post() 或 dispatch() 提交的任务、定时器任务），直到任务队列为空会io_context被停止，为了防止io_context退出，可以在io_context中添加工作对象（boost::asio::executor_work_guard）

> **防止io_context退出**：
> 创建一个线程池，每个线程运行一个 io_context，若 io_context 中的任务队列没有任务，io_context 会退出，但我们希望 io_context 存活
