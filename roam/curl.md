---
tags:
  - linux
  - 命令行工具
  - 计算机网络
---
# curl

**what**：
接口测试，文件下载 工具
- 支持多种协议：`DICT`、`FILE`、`FTP`、`FTPS`、`GOPHER`、`GOPHERS`、`HTTP`、`HTTPS`、`IMAP`、`IMAPS`、`LDAP`、 `LDAPS`、`MQTT`、`POP3`、`POP3S`、`RTMP`、`RTMPS`、`RTSP`、`SCP`、`SFTP`、`SMB`、`SMBS`、`SMTP`、`SMTPS`、`TELNET`、`TFTP`

**how**：基础操作
```bash
# 查看帮助
curl -h category                     # 查看 所有帮助分类（按协议分类）
curl -h <分类>                       # 查看 帮助，eg：http

# 常用操作
curl google.com                      # 发送 GET 请求
curl -X<method> google.com -d <data> # 指定 请求方法，eg：POST
curl -I google.com                   # 查看 响应头
curl -i google.com                   # 查看 响应头+体
curl -v google.com                   # 查看 底层连接信息
curl -L google.com                   # 追踪 重定向
```

**示例**：测试网站 `https://jsonplaceholder.typicode.com/`（`REST` 接口 `CRUD`）
```zsh
curl -XPOST https://jsonplaceholder.typicode.com/posts \
-H 'Content-Type: application/json' \
-d '{
  "title": "测试分类",
  "category": "coding",
  "body": "curl练习",
  "userId": 1
}'

# shell 中 单引号 '：完全原样输出，不会解析
# shell 中 双引号 "：只屏蔽通配符 * ?
```
