---
tags:
  - 工具
  - vscode
---
# vscode

**what**：
跨平台的代码编辑器

## vscode 终端无法识别命令行工具

解决：管理员身份打开vscode

## vscode连接远程服务器

1. 下载 Remote - SSH 插件
2. 命令面板 -> Remote-SSH：添加新的 SSH 主机 添加主机后 活动栏 -> 远程资源管理器 -> 连接
3. 遇到问题，打开 命令面板 -> Remote-SSH：打开 SSH 配置文件 查看

## vscode error：无法与远程服务器建立连接，不满足先决条件

解决 ：下载 低版本vscode，以 arch 为例，在 https://code.visualstudio.com/updates/v1_98 中选择 tarball（arch linux） 下载

**why**：
远程服务器的 GLIBC 版本过低，新版vscode GLIBC 版本较高

## vscode error：连接远程后无法为jupyter选择内核 

在远程服务器上重新安装 jupyter

## vscode error：root权限运行 vscode 但使用的是普通用户配置和扩展

```bash
sudo code --user-data-dir="$HOME/.config/Code" --extensions-dir="$HOME/.vscode/extensions" --no-sandbox
# 只能使用$HOME，使用~不生效。原因未知
```

linux 中 vscode 普通用户配置、扩展插件 位置

|路径|说明|
|---|---|
|~/.config/Code/User/settings.json|普通用户配置|
|~/.vscode/extensions/|扩展插件|