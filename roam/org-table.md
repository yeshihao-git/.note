---
tags:
  - emacs
  - org
---
# org-table

**what**：org-table
org 中用于 操作、计算表格 （能计算，不能合并单元格）

**what**：table.el
org 中用于 生成表格（不能计算，能合并单元格），对应函数为 table-xx

**how**：基础操作

|分类|函数 / 符号|快捷键|作用|
|---|---|---|---|
|视图|org-table-toggle-coordinate-overlays|C-c }|开启表格横纵坐标|
|转换|org-table-convert-region|C-c \||选中区域转为 table（逗号或空格分隔）|
|移动|org-metaup|M-<up>|行上移|
||org-metadown|M-<down>|行下移|
||org-metaleft|M-<left>|列左移|
||org-metaright|M-<right>|列右移|
||org-table-move-cell-up|S-<up>|cell 上移|
||org-table-move-cell-down|S-<down>|cell 下移|
||org-table-move-cell-left|S-<left>|cell 左移|
||org-table-move-cell-right|S-<right>|cell 右移|
|增删|org-shiftmetaup|M-S-<up>|行删除|
||org-shiftmetadown|M-S-<down>|行增加|
||org-shiftmetaleft|M-S-<left>|列删除|
||org-shiftmetaright|M-S-<right>|列增加|
|排序|org-sort|C-c ^|表格排序|
|信息|org-table-field-info|C-c ?|当前 cell 的信息（横纵坐标）|

**how**：表格符号说明

|符号|说明|
|---|---|
|#+TBLFM:|属性（用于计算）|
|@|行|
|$|列|
|..|区域|

## org-table批量移动表格内容

1. 标记需要移动的区域
2. `M-<方向键>`