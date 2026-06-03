---
tags:
  - emacs
  - org
---
# org-roam

**what**：
org 的双链笔记，笔记是以 node 形式存在，笔记之间可以相互关联

**how**：

|函数|快捷键|作用|
|---|---|---|
|org-roam-node-find|C-c n r f|查找 / 新建节点|
|org-roam-node-insert|C-c n r i|关联节点|
|org-roam-buffer-toggle|C-c n r r|显示反向链接节点|

## org-roam设置别名

```org
:PROPERTIES:
:ROAM_ALIASES: <别名>
:END:
```