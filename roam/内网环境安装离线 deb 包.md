---
tags:
  - ubuntu
---
# 内网环境安装离线 deb 包

内网环境中安装外界离线 deb 包，需要保证系统版本和架构一致
```bash
# 查看系统版本和架构
lsb_release -a
uname -m

# 下载 deb 包
sudo apt-get install --download-only build-essential

# 在宿主机安装 deb 包
sudo dpkg -i *.deb

# 修复依赖关系
sudo apt-get install -f
```