---
tags:
  - linux
---
# wget

**what**：
文件下载工具

**why**：
解决命令行环境下的下载需求

## wget下载需要登录才能下载的资源

示例 - 登录 Cityscapes Dataset 网站并下载数据集
```bash
wget --keep-session-cookies --save-cookies=cookies.txt --post-data 'username=myusername&password=mypassword&submit=Login' https://www.cityscapes-dataset.com/login/ # 模拟用户登录：获取认证 Cookie 并存入文件 cookies.txt
wget --load-cookies cookies.txt --content-disposition https://www.cityscapes-dataset.com/file-handling/?packageID=1 # 使用已登录的身份 下载 packageID=1 对应的数据集文件
```

| 参数                         | 作用                                            |
| -------------------------- | --------------------------------------------- |
| --keep-session-cookies     | 保留会话 Cookies（通常是短期有效的身份验证信息）                  |
| --save-cookies=cookies.txt | 将认证后的 Cookie 保存到 cookies.txt 供后续请求使用          |
| --post-data                | 使用 POST 方法提交表单数据（用户名、密码）                      |
| --load-cookies=cookies.txt | 使用之前保存的 Cookie 进行身份认证                         |
| --content-disposition      | 确保使用服务器提供的正确文件名（通常是 Content-Disposition 头部信息） |

# curl

**what**：
网络请求工具

**why**：
模拟命令行环境下 POST/GET/PUT/DELETE 等 HTTP请求，以及 HTTPS、FTP、SCP等多种协议

```bash
curl https://google.com             # 发送GET请求
curl -X <METHOD> https://google.com # 指定请求方法，如:POST
curl -I https://google.com          # 查看响应头
curl -i https://google.com          # 查看响应头+体
curl -v https://google.com          # 查看请求/响应头
```

# tar

**what**：
打包解压工具。tar打包工具 只用于打包多个文件，不压缩文件；gzip/bzip2/xz压缩工具 只用于压缩单个文件，不能打包；因此两者经常结合使用

```bash
tar -xf <压缩包>            # 解压
tar -cf <压缩包> <文件路径> # 打包
```

| 参数  | 作用                            |
| --- | ----------------------------- |
| -x  | 解压                            |
| -c  | 打包                            |
| -f  | 指定文件 (必须放最后)                  |
| -v  | 显示解压详细信息                      |
| -z  | 使用 gzip 压缩 / 解压 .tar.gz 或.tgz |
| -j  | 使用 bzip2 压缩 / 解压 .tar.bz2     |
| -J  | 使用 xz 压缩 / 解压 .tar.xz         |

压缩工具对比

|压缩工具|单独压缩文件后缀|与 tar 结合的归档压缩后缀|速度|压缩率|
|---|---|---|---|---|
|gzip|.gz|.tar.gz / .tgz|快|低|
|bzip2|.bz2|.tar.bz2 / .tbz2 / .tb2|中|中|
|xz|.xz|.tar.xz / .txz|低|高|

# ping

**what**：
网络诊断工具

**why**：
用于测试目标主机是否可达、网络延迟等

**how**：
发送 ICMP请求包（互联网控制报文协议）到目标，接收并统计目标返回的 ICMP应答包，以此来测试网络连通性、延迟等

# bear

**what**：
构建工具的跟踪器（拦截并记录编译过程中的命令和依赖关系），并生成 compile_commands.json （编译数据库），帮助 IDE/静态分析工具 理解如何编译代码，从而提供 代码补全、语法检查、定义跳转等功能

**what**：compile_commands.json
编译数据库，记录编译信息（编译命令、源文件、工作目录等）

**what**：静态分析工具
不运行程序的情况下，通过解析源代码的语法、结构，检查代码问题的工具；常见的有： clang-tidy 、 cppcheck 等

```bash
bear -- <编译/构建命令>
```

## compile_commands.json生成
### cmake项目

方法1 - 修改 CMakeLists.txt
```cmake
# cmake项目根目录中的 CMakeLists.txt 中写入
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
cmake -B build
```

方法2 - 终端中
```bash
# cmake项目根目录中使用以下命令。原因：一个项目中一般有多个CMakeLists.txt，在顶层生成的compile_commands.json包含所有子目录的编译信息
cmake -DCMAKE_EXPORT_COMPILE_COMMANDS=ON .
```

### qmake项目

```bash
# qmake项目根目录中使用以下命令
qmake -r                 # 生成 .qmake.stash
bear -- make - j$(nproc)
```

# nmcli

**what**：
NetworkManager的命令行工具，用于网络配置和管理

**why**：
解决命令行环境下的网络配置和管理需求

## nmcli error：802-11-wireless-security.key-mgmt : property is missing

解决
```bash
nmcli connection up <无线网A>
```

> **问题复现**：
> 无线网A 突然断开，当我使用 =nmcli device wifi connect 无线A password xxx= 再次连接 A 时，出现上述错误

> **原因**：
> NetworkManager 存储了 无线网A网络配置 在 connections，通过 =nmcli connection show= 可以找到 A

# zellij

**what**：
终端复用工具，用于：会话持久化

```bash
zellij attach # 重新连接会话
zellij ls     # 查看所有会话
Ctrl-b d      # 分离会话
```

## zellij一个tab中创建四个对称pane

1. 新建2个pane（此时总共3个pane）
2. 增大pane大小，还原
3. 新建第3个pane


# tr

**what**：
用于 字符替换和删除 的命令

```bash
tr '<原字符集>' '<目标字符集合>' # 字符替换
```

# tree

**what**：
以树状形式展开文件夹结构

```bash
tree -L <层级> <文件夹路径> # （树状形式）查看某个文件夹的结构
```

# rg

**what**：
超快匹配 文本 命令（grep的替代品）

# jq

**what**：
JSON处理 命令（解析、查询、过滤、格式化）

```bash
输出内容 | jq
```

# fd

**what**：
超快查找 文件/目录 命令（find的替代品）

```bash
fd <文件名> <路径>
```

## find

```bash
find <路径> -name "<文件名>"
```

# ss

**what**：
超快 TCP/UDP套接字等 网络连接 查询工具（netstat的替代品）

```bash
sudo ss -tulnp | grep <XX> # 查看端口占用
```

|参数|作用|
|---|---|
|-t|显示 TCP|
|-u|显示 UDP|
|-l|监听端口|
|-n|不要解析域名|
|-p|显示关联的 进程名、pid、fd|

# telnet

**what**：
基于Telnet协议的 远程登录 的命令，常用于测试 TCP端口的连通性

```bash
telnet <主机名/IP> <端口>  # 远程
```

# nslookup

**what**：
DNS分析 命令

```bash
nslookup twitter.com          # 使用 本地默认DNS服务器 查询 twitter.com 对应 IP
nslookup twitter.com 8.8.8.8  # 使用 Google公共DNS服务器

# nslookup <域名> [<DNS服务器>]
```

# nc

**what**：
命令行网络工具，支持：端口扫描、模拟客户端、模拟服务端等
（nc是建连接的，ss是看连接的）

```bash
nc -zv <IP> <端口> # 端口连通性测试
```

|参数|作用|
|---|---|
|-z <IP> <端口>|连通性测试（无数据传输）|
|-v <IP> <端口>|详细信息|

# sed

**what**：
面向文本流的流式编辑器，按 行 处理

**what**：流式编辑器
在输入流（文件或管道）中完成文本处理

# awk

**what**：
处理文本的 编程语言，按 *行和列* 处理

# apt

**what**：
ubunte默认的包管理工具，整合了
1. apt-get
2. apt-config
3. apt-cache

## apt-get设置镜像源

```bash
vim /etc/apt/sources.list.d/ubuntu.sources # 新配置文件位置
vim /etc/apt/sources.list                  # 旧配置文件位置

# 输出以下内容
# 清华源
Types: deb
URIs: http://mirrors.tuna.tsinghua.edu.cn/ubuntu/
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 中科大源
Types: deb
URIs: http://mirrors.ustc.edu.cn/ubuntu/
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 网易164源
Types: deb
URIs: http://mirrors.163.com/ubuntu/
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

# 阿里云
Types: deb
URIs: http://mirrors.aliyun.com/ubuntu/
Suites: noble noble-updates noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

# ps

**what**：
静态 进程状态查看命令；用于查看 命令及对应PID ，后续通过 kill 发送信号
```bash
ps -ef
```

|命令|选项|作用|
|---|---|---|
|ps|-e|显示所有进程|
||-f|显示完整格式（PPID、启动时间等）|
||-H|进程树层级 (进程父子关系)|

# nm

**what**：
符号表信息（函数/变量符号）分析工具，检查库中是否有需要的符号

> **nm与ldd的区别**\
> 使用ldd发现库正确链接，但出现符号错误，此时就需要用nm分析，库版本可能不对

# jobs

**what**：
linux 中用于 后台运行任务 的命令

**how**：

| 命令                  | 作用              |
| ------------------- | --------------- |
| <二进制文件> &           | 后台运行            |
| jobs -l             | 显示作业的 PID 和详细信息 |
| fg %<作业号>           | 后台作业切换到前台运行     |
| bg %<作业号>           | 已停止作业恢复到后台运行    |
| kill -<数字> %< 作业号 > | -20 为停止后台作业     |
