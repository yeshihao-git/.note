---
tags:
  - 工具
---
# clash verge rev

**what**：clash
网络代理工具

**what**：clash verge
基于 clash内核 的代理客户端

**what**：clash verge rev
clash verge 的分支重构版

机场:
[山海](https://shanhai.me/#/dashboard)
[魔戒](https://mojie.app/login)
[百合电信](https://www.yuritele.com/auth/login)

## clash同一局域网内连另一台机器的代理

1. 机器A 和 机器B 在 同一局域网
2. 在 机器A 中的 代理 clash verge 中 打开局域网连接
3. 机器B 以 linux终端走代理 的方式连接
	 - IP  ：机器A的IP
	- PORT：clash verge 的端口

## 关闭 clash 系统代理后退出，QQ能上网但浏览器无法上网问题

1. 右键 网络图标，选择 **网络和 Internet 设置**
![[Pasted image 20260430172703.png|308]]

2. 选择**更改适配器选项**
![[Pasted image 20260430172728.png|229]]

3. 右键正在使用的网络，选择**属性**
![[Pasted image 20260430172816.png|449]]

4. 双击 **Internet 协议版本 4（TCP/IPv4）**
![[Pasted image 20260430172854.png|322]]

5. 更改**首选 DNS 服务器**，这里使用阿里云的 223.5.5.5
![[Pasted image 20260430172927.png|325]]