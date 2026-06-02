---
tags:
  - 操作系统
---
# socket

**what：**
套接字，通过系统调用 socket() 创建并返回 用于网络通信的文件描述符[^1]即 fd，后续网络通信的操作都是基于此 fd

## socket 编程的基本流程 

- **服务端**： socket ：创建并返回用于网络通信的 fd -> bind ：绑定 socket 到本地IP端口 -> listen ：监听 -> accept ：接受连接，返回新的socket的文件描述符 -> recv/send ：收发数据
- **客户端**： socket -> connect -> send/recv

## socket 相关的数据结构
### addrinfo 结构体

**what：**
配置信息去查询内容，返回的实际内容在 sockaddr（调用 getaddrinfo() 填充该结构体，并以链表形式返回）
```c
struct addrinfo {
    int              ai_flags;     // AI_PASSIVE, AI_CANONNAME, etc.
    int              ai_family;    // AF_INET, AF_INET6, AF_UNSPEC（设置IPv4、IPv6、任意）相当于是我要求返回的协议族，实际返回的协议族在 sockaddr中
    int              ai_socktype;  // SOCK_STREAM, SOCK_DGRAM
    int              ai_protocol;  // use 0 for "any"
    size_t           ai_addrlen;   // size of ai_addr in bytes
    struct sockaddr *ai_addr;      // struct sockaddr_in or _in6
    char            *ai_canonname; // full canonical hostname

    struct addrinfo *ai_next;      // 指向下一个链表节点
};
```

### sockaddr 结构体

**what：**
通用地址结构体；保存套接字地址信息，但是 sa_data 是 `char[14]` 不够直观，因此使用 sockaddr_in、sockaddr_in6 替代。可以和 struct sockaddr_in* 互转
```c
struct sockaddr {
    unsigned short    sa_family;    // address family, AF_xxx（设置IPv4、IPv6、任意）
    char              sa_data[14];  // 14 bytes of protocol address（套接字目标地址和端口号）
};
```

### sockaddr_in 结构体

**what：**
IPv4 专用地址结构体，比 sockaddr 更直观
```c
struct sockaddr_in {
    short int          sin_family;  // Address family, AF_INET（协议族）
    unsigned short int sin_port;    // Port number（端口）使用 htons转网络字节序
    struct in_addr     sin_addr;    // Internet address（IP地址）
    unsigned char      sin_zero[8]; // 用于填充该结构体的长度：与 struct sockaddr 一致，使用 memset() 设置全0
};

// Internet address (a structure for historical reasons)
struct in_addr {
    uint32_t s_addr; // that's a 32-bit int (4 bytes)
};
```

### sockaddr_in6 结构体

**what：**
IPv6 专用地址结构体，比 sockaddr 更直观
```c
struct sockaddr_in6 {
    u_int16_t       sin6_family;   // address family, AF_INET6
    u_int16_t       sin6_port;     // port, Network Byte Order
    u_int32_t       sin6_flowinfo; // IPv6 flow information
    struct in6_addr sin6_addr;     // IPv6 address
    u_int32_t       sin6_scope_id; // Scope ID
};

struct in6_addr {
    unsigned char   s6_addr[16];   // IPv6 address
};
```

## epoll 相关的数据结构
### epoll_event

epoll事件的数据结构
```c
typedef union epoll_data
{
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;

struct epoll_event
{
  uint32_t events;    // 关注的事件类型
  epoll_data_t data;  // 用户数据
} __EPOLL_PACKED;
```

关注的事件类型：
- **EPOLLIN**：数据可读
- **EPOLLOUT**：数据可写
- **EPOLLERR**：错误条件
- **EPOLLHUP**：连接完全断开（读写都断开）
- **EPOLLRDHUP**：对端写关闭
- **EPOLLPRI**：紧急数据
- **EPOLLET**：边缘触发模式（默认是水平触发）
- **EPOLLONESHOT**：单次触发后禁用监控

## 字节序转换

```c
htons()  // 主机到网络short格式
htonl()  // 主机到网络long格式
ntohs()  // 网络到主机short格式
ntohl()  // 网络到主机long格式
```


[^1]:   **文件描述符**：与打开文件关联的整数。这个文件可以是
	1. 网络连接
	2. 管道
	3. 终端
	4. ...
