---
tags:
  - linux
---
# arch linux

**what**：
linux 的一个发行版
1. systemd 作为默认的 init系统 与 服务管理器
2. pacman 作为默认的包管理器

## arch linux安装向日葵

```bash
yay -S sunloginclient
sudo systemctl start runsunloginclient.service
sudo systemctl enable runsunloginclient.service
```

## arch linux终端musicfox播放音乐时无法使用耳机播放 

```bash
pacman -S pipewire-alsa
```

## arch设置中文环境 

1. 生成语言包
```bash
sudo vim /etc/locale.gen
# 取消 zh_CN.UTF-8 UTF-8 前的注释
sudo locale-gen # 生成语言包
```
2. 设置 LANG 环境变量（可能出现终端豆腐块问题，解决见 wiki）
```bash
sudo vim /etc/locale.conf
LANG=zh_CN.UTF-8
```
3. reboot

## arch安装linux-zen内核

```bash
sudo pacman -S linux-zen linux-zen-headers # 安装 linux-zen
sudo grub-mkconfig -o /boot/grub/grub.cfg  # 更新引导加载程序
reboot
# 引导菜单 -> 高级菜单 -> linux-zen
# 验证内核：uname -r
```

## arch linux安装

1. 下载官方镜像，将官方镜像烧录到 U盘 中（win中推荐使用Rufus），此时 U盘 中就有一个完整的 arch linux live系统
2. live系统可以理解为临时的系统，因为当我们将 U盘 插入主机中的时候，在 固件（BIOS、UEFI）中选择从 usb启动 就会将 U盘 中的完整 arch linux live系统 加载到内存中运行
3. 然后使用 chroot 切换根目录到主机的硬盘中，就能在主机的硬盘中进行各种操作了
4. arch linux 不支持 Secure Boot，UEFI 默认启动 Secure Boot，因此我们需要在主板中将其关闭
5. 关闭 windows 的快速启动和休眠；方法见： windows关闭快速启动和休眠

## arch linux管理java jdk

archlinux-java 用于管理和切换系统中已安装的java版本
```bash
archlinux-java status                      # 查看所有jdk版本
sudo archlinux-java set <java-environment> # 设置java的jdk版本
```

# pacman

**what**：
arch linux 和 windows上msys2 的默认包管理工具

改镜像
```bash
vim /etc/pacman.conf
```

包的 安装/卸载/查询/升级降级
```bash
pacman -S <包>  # 安装包
pacman -Ss <包> # 远程搜索包（模糊搜索）
pacman -Sy      # 更新同步数据库（同步数据库：/var/lib/pacman/sync）
pacman -Syu     # 更新同步数据库、升级所有已安装软件

pacman -R <包>  # 删除软件
pacman -Rn <包> # 删除软件+配置

pacman -Q <包>  # 本地搜索包（精确搜索）（本地数据库：/var/lib/pacman/local）
pacman -Qs <包> # 本地搜索包（模糊搜索）
pacman -Ql <包> # 列出包中的所有文件（包含头文件、库文件等）

pacman -U <包>  # 用包缓存目录中的包升级降级包（包缓存目录：/var/cache/pacman/pkg）
```

## arch linux的wps乱码问题

给freetype降级
```bash
pacman -U /var/cache/pacman/pkg/freetype2-2.13.0-1-x86_64.pkg.tar.zst
```

# yay

**what**：
pacman 和 AUR助手 的前端工具，整合了 官方软件仓库（pacman 功能）和 Arch用户仓库（AUR），支持一键搜索、安装、更新 官方软件包 和 AUR社区软件包

## yay安装

1. 挂代理；见 同一局域网一台机器连另一台机器的代理
2. git clone https://aur.archlinux.org/yay-bin.git
3. cd yay-bin
4. makepkg -si
5. 有时候 makepkg -si 因为网络问题没法下载 yay_12.5.0_x86_64.tar.gz 导致失败；此时在 yay-bin文件夹中 wget https://github.com/Jguer/yay/releases/download/v12.5.0/yay_12.5.0_x86_64.tar.gz 再执行 makepkg -si
6. 也可以在 浏览器中手动设置代理 再去 yay 的 github 上下载 yay_12.5.0_x86_64.tar.gz

## yay error：proxyconnect tcp:EOF

1. 问题在于yay 底层是通过 curl 访问走代理（proxyconnect）连接 AUR服务器（https://aur.archlinux.org ，可以通过 curl -v https://aur.archlinux.org 看详细连接过程） ，而我在终端输入：export https_proxy="https://127.0.0.1:7890" 希望 curl 走HTTPS的代理，  但 clash 是 HTTP代理软件 ，只能通过 http://127.0.0.1:7890 访问
2. 因此终端设置：export https_proxy="http://127.0.0.1:7890" （ 再次注意 ：这里是 http:// ！） 

# systemd

**what**：
systemd 是 arch 默认的init进程（PID 1）；配套了一系列的命令行工具：
1. systemctl ：服务管理
2. journalctl：日志管理
3. networkctl：网络管理
4. loginctl  ：登录管理

## systemctl

**what**：
用于管理系统服务
