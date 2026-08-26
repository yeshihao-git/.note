---
tags:
  - 工具
  - 前端
---
# npm

**what**：
node.js 的包管理工具，安装 node.js 时会自动安装 npm，推荐使用 nvm 安装 node.js

## nvm

**what**：
（Node Version Manager）用于安装 node.js，可以自由切换多个 node.js 版本，具体见：<https://nodejs.org/en/download>

**how**：

|功能分类|命令|说明|
|---|---|---|
|**版本管理**|`nvm --version`|查看 nvm 自身版本|
||`nvm list` 或 `nvm ls`|列出所有已安装的 Node.js 版本|
||`nvm list available` 或 `nvm ls-remote`|列出所有可安装的远程版本|
|**安装版本**|`nvm install <版本号>`|安装指定版本，如 `nvm install 20.0.0`|
||`nvm install --lts`|安装最新的 LTS（长期支持）版本|
||`nvm install node`|安装最新的稳定版本|
|**切换版本**|`nvm use <版本号>`|**临时切换**：在当前终端会话中使用指定版本|
||`nvm alias default <版本号>`|**永久切换**：设置默认使用的版本|
||`nvm current`|查看当前正在使用的版本|
|**卸载版本**|`nvm uninstall <版本号>`|卸载指定版本|
|**其他实用命令**|`nvm which <版本号>`|显示指定版本的安装路径|
||`nvm run <版本号> <文件>`|使用指定版本直接运行脚本|
||`nvm reinstall-packages <版本号>`|重装指定版本的全局 npm 包到当前版本|
||`nvm deactivate`|取消当前 nvm 的影响，恢复使用系统默认 Node.js|
