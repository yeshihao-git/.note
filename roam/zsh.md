---
tags:
  - 工具
---
# zsh

**what**：
命令行解释器（shell）的一种

> **终端、命令行解释器、操作系统内核 之间的关系**：
> - 终端：人机交互窗口，只做输入输出展示
> - shell：程序，解析终端传来的指令，内置命令自行处理，外部命令调用系统 API，交给内核处理
> - 操作系统内核：执行指令后将结果返回给 shell，再由 shell 返回给终端

## zsh 配置

1. 设置终端和 git 走代理：[[代理设置#linux 终端走代理|linux 终端走代理]]、[[代理设置#git 走代理|git 走代理]]
2. 安装基本工具
```zsh
# 更新软件源
sudo apt update && sudo apt upgrade -y

# 安装 zsh git curl
sudo apt install zsh git curl -y
```
3. 设置默认 shell 为 zsh
```zsh
chsh -s /bin/zsh
```
4. 安装 oh-my-zsh
```zsh
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
5. 安装插件：zsh-autosuggestions、zsh-syntax-highlighting
```zsh
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting 
```
6. 编辑 .zshrc 开启插件
```zsh
plugins=(
	git
    zsh-autosuggestions
    zsh-syntax-highlighting
	z
	extract
)

# 代理设置
export http_proxy="http://127.0.0.1:7890"
export https_proxy="http://127.0.0.1:7890"

git config --global http.proxy "http://127.0.0.1:7890"   
git config --global https.proxy "http://127.0.0.1:7890"  
```