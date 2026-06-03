---
tags:
  - emacs
  - org
---
# org-noter

**what**：
org 中记PDF笔记的插件

**how**：

|函数|作用|
|---|---|
|org-noter-insert-note|当前页面插入笔记|
|org-noter-insert-precise-note|鼠标选中区域插入笔记|
|org-noter-insert-note-toggle-no-questions|鼠标选中区域变成 headline 插入笔记|
|org-noter-sync-prev-note|将 PDF 翻到上一个笔记处|
|org-noter-sync-next-note|将 PDF 翻到下一个笔记处|
|org-noter-sync-current-note|从笔记定位 PDF 位置|

## org-noter设置PDF选中区域高亮

变量 org-noter-highlight-selected-text 设置为 on

## org-noter设置笔记对应的PDF以及跟踪PDF页数

1. M-x org-noter -> 选择 PDF文件
2. 生成 drawer
```org
#-------- 设计模式.org---------#
1 :PROPERTIES:
2 :NOTER_DOCUMENT: /home/yoshiki01/Documents/(软件工程)设计模式v1.31（全部）.pdf      # org文件对应的pdf
3 :NOTER_PAGE: 202                                                                    # 打开pdf时的页数
4 :END:
5 #+title: 设计模式
#-------- 设计模式.org---------#
```
   #+end_example

