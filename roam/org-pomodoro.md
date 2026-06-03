---
tags:
  - emacs
  - org
---
# org-pomodoro

**what**：
org 中的番茄钟插件，基于 org-clock 的 org-clock-in 和 org-clock-out 函数实现

**how**：
1. 开启番茄钟： M-x org-pomodoro
2. 关闭番茄钟： M-x org-pomodoro
3. 生成任务报告： M-x org-clock-report
```
:LOGBOOK:
#+BEGIN: clocktable :scope subtree :maxlevel 2
#+CAPTION: Clock summary at [2024-12-22 日 12:32]
| Headline   | Time |
|------------+------|
|*Total time*|*0:06*|
|------------+------|
| tets       | 0:06 |
#+END:
CLOCK: [2024-12-22 日 12:21]--[2024-12-22 日 12:26] =>  0:05    // CLOCK: <开始时间>--<结束时间> => <持续时间>
CLOCK: [2024-12-22 日 12:18]--[2024-12-22 日 12:19] =>  0:01
:END:
```

|函数|快捷键|作用|
|---|---|---|
|org-pomodoro|C-c t t|开启 / 关闭番茄钟|
|org-pomodoro（org 文件外使用）|C-c t t|查看当前和历史番茄钟|
|org-pomodoro-extend-last-clock||延长上一个番茄钟|

## org-pomodoro设置手动取消番茄钟也能记录时间

1. M-x customize
2. Org Pomodoro Keep Killed Pomodoro Time -> on