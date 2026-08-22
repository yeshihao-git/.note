---
tags:
  - 工具
  - git
---
# gitea

**what**：
代码托管平台，免费私有化部署

**how**：
配置位置：`/opt/gitea/data/gitea/conf/app.ini`
配置说明：https://docs.gitea.com/zh-cn/administration/config-cheat-sheet/

## 从另一个 gitea 迁移仓库到当前 gitea

1. 在==当前 gitea== 的 app.ini 中写入，允许从外部 gitea 迁移
```ini
[migrations]
ALLOW_LOCALNETWORKS = true
ALLOWED_DOMAINS = 10.8.254.129（外部的IP）
BLOCKED_DOMAINS =
```

2. 在当前 gitea 中
![[Pasted image 20260822140213.png]]
![[Pasted image 20260822140333.png]]
![[Pasted image 20260822140447.png|486]]