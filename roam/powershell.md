---
tags:
  - 工具
---
# powershell 个人配置

1. `PSReadLine`： https://github.com/PowerShell/PSReadLine ，查看`Installation and Upgrading`部分
2. 设置emacs键位：
```powershell
notepad $PROFILE
Set-PSReadLineOption -EditMode Emacs
```

3. 使用vim作为终端编辑器
```powershell
winget install vim.vim
notepad $PROFILE
Set-Alias vim "<vim路径>"
```

# powershell 常用命令

```powershell
###=== 获取帮助 ===###
get-help 

###=== 文件与目录 ===###
ii .                           # 打开资源管理器
mv                             # 移动和重命名
cp                             # 复制
rm                             # 删除
cat                            # 查看文件内容
echo                           # 打印 
rg                             # 快速搜索（scoop install ripgrep）

###=== 网络 ===###
ipconfig                       # IP 配置
netstat -ano | findstr <:端口> # 查看网络端口
ping                           # 连通性
curl                           # 网页请求

###=== 进程 ===###
ps                             # 打印进程列表
kill                           # 杀死进程

###=== 服务 ===###
sc                             # 服务相关命令
```

