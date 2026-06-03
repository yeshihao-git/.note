---
tags:
  - emacs
---
# doom emacs

**what**：
[[emacs]] 的一个配置框架。doom 内部集成了大量的插件且对大量函数进行了二次封装
1. doom/开头的函数：代表 doom核心功能
2. +开头的函数：代表 模块的扩展功能

**how**：
```bash
doom sync     # 同步配置
doom help     # 查看帮助
```

**how**：doom配置文件
在 .doom.d 或 .config/doom 中（ doom help 查看），内含配置文件如下：

|配置文件|作用|
|---|---|
|init.el|插件配置|
|config.el|用户配置（doom 启动时加载，也能 C-h r r 重新加载）|
|custom.el|customize 自动生成的配置（不手动修改）|
|packages.el|额外包配置|

**how**：doom安装位置
在 .config/emacs 或 .emacs.d 中（ doom help 查看）

**how**：doom帮助
除了 emacs 中的相关帮助，doom 也封装了很多 帮助函数

|函数|说明|
|---|---|
|doom/help|doom emacs 文档|
|doom/help-xxx|各类帮助|
|doom/describe-xxx|各类描述|

## doom使用rg搜索时预览内容

`M-x +default/search-project` 进行rg搜索时，使用 `C-SPC` 预览内容

## windows中安装doom emacs（msys2版）
### 安装前置要求

- [[git]]
- ripgrep：安装 windows-gnu版本
- fd：安装 windows-gnu版本

### 安装原始emacs

在 msys2 ucrt64 中
1. pacman -S mingw-w64-x86_64-emacs

### 设置环境变量

windows 中设置环境变量
1. 用户变量 设置 HOME 为 `C:\Users\<你的用户名>`
2. 用户变量 Path 中添加
	- `C:\msys64\mingw64\bin`
	- `C:\<ripgrep路径>`
	- `C:\<fd路径>`
	- `C:\<doom命令路径>`
3. 用户变量 设置 HTTP_PROXY、HTTPS_PROXY
4. 重启终端

### 安装 doom emacs 

这里选择将 doom emacs 安装到 msys2 的 HOME 中，使用 powershell
1. `cd C:\msys64\home\<你的用户名>\`
2. `git clone https://github.com/hlissner/doom-emacs .emacs.d`
3. `doom install`
4. `doom sync`
   
检查并安装缺失的依赖   
1. `doom doctor`

删除 windows HOME目录中的 .emacs.d 
1. `rm C:\Users\<你的用户名>\.emacs.d`

cmd 中 创建 软链接 链接 msys2 中 .emacs.d（powershell 中运行 emacs，会自动查找 HOME目录中的 .emacs.d，但 .emacs.d 安装在 msys2 中，因此需要软链接）
1. `mklink /j "C:\Users\<你的用户名>\.emacs.d" "C:\Users\USER\AppData\Roaming\.emacs.d"`

启动 emacs，安装 图标
1. `M-x nerd-icons-install-fonts`
2. 下载后，手动安装 字体文件

windows 中安装 org-download 前置依赖
1. `scoop install imagemagick`
2. 重启终端和 emacs

### 创建 emacs 快捷方式

1. 鼠标右键 -> 新键快捷方式
2. 链接路径 `C:\msys64\mingw64\bin\runemacs.exe`
runemacs.exe 在启动 emacs 时，不会启动终端。emacs.exe 则会 