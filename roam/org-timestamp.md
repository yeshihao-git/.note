---
tags:
  - emacs
  - org
---
# org-timestamp

**what**：
org 的时间戳功能，用于在 org文件 中 插入、操作日期

| 函数                     | 快捷键   | 作用       |
| ---------------------- | ----- | -------- |
| org-timestamp          | C-c . | 插入 激活时间  |
| org-timestamp-inactive |       | 插入 非激活时间 |

**what**：激活时间
 用 `<>` 表示，会显示在 Org Agenda视图
 
**what**：非激活时间
 用 `[]` 表示，不会显示在 Org Agenda视图
 
```bash
<2024-12-22 日 +1d>    # 激活时间
[2024-12-22 日 17:07]  # 非激活时间
```

## org-timestamp设置重复任务

|标记|作用|
|---|---|
|+1d|多次任务，全部完成 -> 所有任务 done|
|.+1d|多次任务，完成一次 -> 所有任务 done|