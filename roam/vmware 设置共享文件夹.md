---
tags:
  - 工具
---
# vmware 设置共享文件夹

虚拟机 - 设置 - 选项 - 共享文件夹
```bash
# 手动挂载
sudo mkdir -p /mnt/hgfs
sudo vmhgfs-fuse .host:/ /mnt/hgfs -o allow_other
```