---
tags:
  - emacs
  - org
---
# org-agenda

**what**：
org mode 的 任务和日程管理 系统。 Org Agenda视图 ，用于查看、管理：待办事项 (TODO) 、日程 (SCHEDULED)、截止日期 (DEADLINE) 等信息

**how**：
1. M-x org-agenda 启动 agenda菜单
2. 将文件加入 Org Agenda视图 以进行显示/管理
   
|函数|快捷键|作用|
|---|---|---|
|org-agenda-file-to-front|C-c [|将文件加入 org-agenda-files|
|org-remove-file|C-c ]|将文件移出 org-agenda-files|
   - **what**：org-agenda-files（变量）
     org-agenda 管理的文件列表
1. Org Agenda视图 的相关操作
   
| 分类   | 快捷键                | 作用                          |
| ---- | ------------------ | --------------------------- |
| 日期跳转 | .                  | 今天                          |
|      | j                  | 从 Calendar视图 跳转             |
|      | f                  | 下一周                         |
|      | b                  | 上一周                         |
| 任务   | t                  | TODO 菜单                     |
|      | ,                  | 优先级                         |
|      | >                  | org-schedule                |
|      | :                  | tags                        |
|      | e                  | effort                      |
|      | z                  | 添加笔记                        |
|      | C-k                | 删除任务及 org 文件 对应部分           |
|      | C-c C-w            | org-refile                  |
|      | SPC                | 另一个 buffer 显示 任务对应 org 文件   |
|      | TAB                | 同上，但光标移到另一个 buffer 中        |
|      | F                  | （跟随模式）光标在任务中移动，显示对应 buffer  |
|      | o                  | 关闭 _Org Agenda_视图 外的 buffer |
| 计时   | I                  | 开始 / 结束时钟                   |
|      | O                  | 结束时钟                        |
|      | X                  | 结束正在运行的时钟                   |
| 视图   | v                  | 切换视图                        |
|      | r 或 g              | 重建视图                        |
|      | org-agenda-columns | 打开 column view              |
| 视图过滤 | <                  | 类别（category）                |
|      | \|                 | 标签（tags）                    |
|      | -                  | 努力值（effort）                 |
|      | =                  | 正则（regexp）                  |
|      | \|                 | 取消一切过滤操作                    |
|      | /                  | 组合过滤                        |

## org-agenda视图中进行过滤

1. Org Agenda视图 中按 / （调用的是 org-agenda-filter）
2. 语法： (+/-)category(+/-)tags(+/-)effort(+/-)regexp
```
+work-John+<0:10-/plot / # 选择category为work，反选tags为John，选择effort小于10分钟，反选带有plot关键字 的任务
```

## org-agenda视图中切换视图

1. Org Agenda视图 中按 v （调用的是 org-agenda-view-mode-dispatch）

|视图种类|
|---|
|日 / 周 / 月 / 年视图|
|Log 模式|
|周报|
|diary 模式（显示各种节日）|
|时间打卡完整性|

## org-agenda error：视图中子任务完成时todo颜色不变

1. M-x customize
2. 在 Org Agenda Dim Blocked Tasks 中设置 Do not dim

## org-agenda error：重复任务过了deadline依旧在视图中显示

解决：设置变量 org-agenda-skip-scheduled-repeats-after-deadline 为 t

> **问题复现**：
> 25-27号为重复任务，SCHEDULED 设置为25，DEADLINE 设置为27，但27号后依旧显示重复任务
