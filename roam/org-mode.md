---
tags:
  - emacs
  - org
---
# org-mode

**what**：
是 emacs 中的一个主模式，也是集多功能于一身的纯文本系统

## 基础操作

|分类|快捷键|作用|
|---|---|---|
|视图|TAB|循环显示 headline 层级|
||C-<数字> S-TAB|显示 headline <数字> 层级|
||C-c C-k|只显示 headline|
||C-u TAB|显示 1 层级的 headline|
||C-u C-u TAB|显示 刚进入 org 文件时的 headline 层级|
||C-u C-u C-u TAB|显示 全展开的 entry（包括 drawer）|
|移动|C-c C-b|移动到 上个同级 headline|
||C-c C-f（按键被占用）|移动到 下个同级 headline|
||C-c C-u|移动到 上级的 headline|
|编辑|C-S-RET|上方插入 同级 headline|
||C-RET|下方插入 同级 headline|

**what**：outline tree
大纲树（org mode 以 outline tree 的形式组织文档）

**what**：headline
标题（outline tree 由 headlines 构成）

**what**：entry
标题 + 内容

**what**：drawer
抽屉
```org
:PROPERTIES:
:key1: value1
...
:END:
```

## 工具链

| 类别  | 作用         | 工具                          |
| --- | ---------- | --------------------------- |
| 任务  | 任务、日程管理    | [[org-agenda]]              |
| 笔记  | 双链笔记       | [[org-roam]]                |
|     | PDF 笔记     | [[org-noter]] [[pdf-tools]] |
|     | 导出笔记       | [[org-pandoc]]              |
|     | 捕获笔记       | [[org-capture]]             |
|     | 移动笔记       | [[org-refile]]              |
|     | 粘贴图片       | [[org-download]]            |
|     | 文学编程       | [[org-babel]]               |
| 表格  | 表格操作、计算、生成 | [[org-table]] table.el      |
| 时间  | 番茄钟        | [[org-pomodoro]]            |
|     | 计时         | [[org-clock]]               |
|     | 时间戳        | [[org-timestamp]]           |
| 视图  | 跳转         | [[org-goto]]                |
|     | 列视图        | [[org-columns]]             |
| 杂项  | 创建 id      | [[org-id]]                  |

## org文件中设置首次进入时显示的headline层级

**文件级别**：在 org 文件的上方写入
```org
#+STARTUP: overview
#+STARTUP: content
#+STARTUP: showall
#+STARTUP: show<数字>levels
#+STARTUP: showeverything（默认）
```
#+end_example

**标题级别**：在 drawer 里写入:VISIBILITY:键值对
```org
 * Heading 1
:PROPERTIES:
:VISIBILITY: show2levels
:END:
```

## org设置显示headline数字

M-x org-num-mode

## org隐藏headline前的*和强调标记

1. M-x customize
2. Org Hide Leading Stars    -> on
   Org Hide Emphasis Markers -> on

## org设置标签（tag）

给 文件 设置标签     ：文件开头 `#+filetags: tag1 tag2 ...`
给 headline 设置标签 ：文件开头 `#+TAGS: tag1(快捷键1) tag2(快捷键2) ...` ，此时在 headline 上 C-c C-c 可以使用标签， (快捷键1) 为可选

