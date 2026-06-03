---
tags:
  - emacs
  - org
---
# org-pandoc

**what**：org-pandoc
org-export的扩展增强导出插件，基于 org-export 实现

**what**：org-export
org 内置的导出引擎，用于将 org文件 导出为多种格式

**how**：

|函数|作用|
|---|---|
|org-pandoc-export-to-<文件类型>|导出 <文件类型> 文件|
|org-pandoc-export-as-<文件类型>|预览 <文件类型> 文件|

## org设置导出目录

```org
# -------DoomEmacs使用手册.org---------#
1 ...
2 #+EXPORT_FILE_NAME: ~/.org/export/DoomEmacs使用手册
3 ...
# -------DoomEmacs使用手册.org---------#
```

## org导出思维导图

1. M-x org-freemind-export-to-freemind -> 生成.mm文件
2. 用 freeplane 打开 .mm文件

## org导出markdown

1. M-x org-pandoc-export-to-markdown_mmd （markdown推荐）