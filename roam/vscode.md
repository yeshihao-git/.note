---
tags:
  - 工具
  - vscode
---
# vscode

**what**：
跨平台的代码编辑器

## vscode 编译调试 C++ 项目
### gcc 编译 C++ 项目

0. `vscode` 中安装 `C++` 相关插件  
![[Pasted image 20260612174136.png|292]]

1. 配置 `IntelliSence`：`Ctrl +Shift + p` 打开命令面板  
![[Pasted image 20260410142041.png|449]]  
在包含路径中输入头文件路径  
![[Pasted image 20260410142427.png]]

2. 配置 `tasks.json`：`vscode` 右上角选择 “运行 `C/C++` 文件”  
![[Pasted image 20260410143125.png]]  
![[Pasted image 20260410143130.png]]  
此时出现错误，因为我们还没进行头文件路径执行、链接库等操作  
![[Pasted image 20260410143135.png]]  
打开 `tasks.json`  
![[Pasted image 20260410143411.png]]  
指定头文件路径，链接库  
![[Pasted image 20260410143543.png]]  
重新点击`vscode` 右上角 “运行 `C/C++` 文件”  

> **tasks.json**：
> 编译任务文件，`vscode` 编译代码用 配置文件
> 
> **c_cpp_properties.json**：
> `C++` 编辑配置文件（配置 `IntelliSence`时的相关信息就在这里）

### gdb 调试 C++ 项目

`vscode` 新版 `C/C++` 插件无需手写 `launch.json` 就能一键调试了
![[Pasted image 20260410155840.png]]
1. 打断点

2. 点击右上角 “调试 C/C++ 文件”

3. 上方出现调试面板：继续、逐过程、单步调试、单步跳出、重启、停止

4. 左侧用于查看：局部变量、监视变量变化、调用栈信息、断点信息等

5. 下方为终端输出，用于查看：日志信息

> **launch.json**：
> 调试任务文件，`vscode` 调试代码用 配置文件

## vscode 终端无法识别命令行工具

解决：管理员身份打开vscode

## vscode 连接远程服务器

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