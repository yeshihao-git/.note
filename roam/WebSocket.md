---
tags:
  - 计算机网络
---
# WebSocket

**what：**
全双工 应用层协议（HTTP 是半双工），实现客户端和服务器双向实时通信，适用高频信息交互场景（eg：网页游戏、网页聊天室）

**why：** 
虽然 TCP 是全双工，但未解决粘包问题；WebSocket 解决了，报文格式是 消息头+消息体

> **半双工**：同一时间，只能一方发送数据
> **全双工**：同一时间，双方都能发送数据

## 建立 WebSocket 通信流程

![[Pasted image 20260519164436.png|391]]
1. TCP三次握手
2. 浏览器 先统一使用 HTTP
   - 普通HTTP请求：后续继续HTTP通信
   - 想建立WebSocket连接：HTTP请求头中有以下字段
```json
Connection: Upgrade                               /* 浏览器希望升级协议 */
Upgrade: WebSocket                                /* 希望升级为 WebSocket */
Sec-WebSocket-Key: T2a6wZlAwhgQNqruZ2YUyg==\r\n   /* 随机生成的base64码 */
```

3. 服务器返回 状态码为101的报文（含有 用公开算法将客户端base64码生成的字符串）
```json
/* 101：转换协议的状态码 */
HTTP/1.1 101 Switching Protocols\r\n                
/* 公开算法 将 客户端的Sec-WebSocket-Key生成的另一个字符串 */
Sec-WebSocket-Accept: iBJKv/ALIW2DobfoA4dmr3JHBCY=\r\n 
Upgrade: WebSocket\r\n
Connection: Upgrade\r\n
```

4. 浏览器用 公开算法 将 base64 码转换为字符串，与服务器返回的 字符串比对一致则通过，开始通信