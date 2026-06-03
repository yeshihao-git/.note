---
tags:
  - emacs
  - org
---
# org-columns

**what**：
列视图，方便查看/编辑 outline tree 中的属性

|函数|作用|
|---|---|
|org-columns|打开 column view|
|org-columns-quit|退出 column view|

## org mode自定义column view格式

```bash
org文件开头输入以下内容
#----------------.org文件------------------#
#+COLUMNS: [%列宽度][标题/属性] x n
#----------------.org文件------------------#
```
