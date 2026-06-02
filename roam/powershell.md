---
tags:
  - 工具
---
# 设置emacs键位

1. `PSReadLine`： https://github.com/PowerShell/PSReadLine ，查看`Installation and Upgrading`部分
2. 永久设置：
```powershell
notepad $PROFILE
Set-PSReadLineOption -EditMode Emacs
```

# 打开资源管理器

```powershell
ii .
```

# 使用 vim

```powershell
winget install vim.vim
notepad $PROFILE
Set-Alias vim "<vim路径>"
```