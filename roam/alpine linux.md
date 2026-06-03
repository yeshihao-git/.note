---
tags:
  - linux
---
# alpine linux

**what**：
轻量级linux发行版，以小巧（约5MB）和高效著称，包管理器为 apk ；广泛用于 docker基础镜像

## apk设置镜像源

```bash
vi /etc/apk/repositories  # 编辑镜像源文件。软件包缓存目录：/var/cache/apk/*
#---------------------------------------------------------#
# 阿里云镜像源
https://mirrors.aliyun.com/alpine/v3.14/main
https://mirrors.aliyun.com/alpine/v3.14/community
# 清华镜像源
https://mirror.tuna.tsinghua.edu.cn/alpine/v3.14/main
https://mirror.tuna.tsinghua.edu.cn/alpine/v3.14/community
#---------------------------------------------------------#
apk update                # 更新包索引
```