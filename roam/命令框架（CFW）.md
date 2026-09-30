---
tags:
  - 软件架构
  - 设计模式
aliases:
  - 命令模式
---
# 命令框架（CFW）

**what**：
用于实现 undo / redo 

**how**：
统一的命令接口 + 双栈管理器

**how**：**核心要素一**：统一的命令接口 (Command Interface)
每一个可以被撤销的操作，都会被封装成一个具体的类（对象）。它必须实现三个方法：

```python
class Command:
    def execute(self):   # 执行操作
        pass
    def undo(self):      # 撤销操作（回滚到执行前的状态）
        pass
    def redo(self):      # 重做操作（通常等同于 execute()）
        pass
        
# 具体的命令继承这个接口类 Command
```

**how**：**核心要素二**：双栈管理器 (History / Stack Manager)
管理器内部维护两个列表（栈）：`Undo栈` 和 `Redo栈`。

```python
class UndoManager:
    def __init__(self):
        self.undo_stack = []
        self.redo_stack = []

    def execute_command(self, command):
        command.execute()
        self.undo_stack.append(command)
        self.redo_stack.clear() # 执行新操作，清空重做栈

    def undo(self):
        if not self.undo_stack:
            print("没有可撤销的操作")
            return
        command = self.undo_stack.pop()
        command.undo()
        self.redo_stack.append(command)

    def redo(self):
        if not self.redo_stack:
            print("没有可重做的操作")
            return
        command = self.redo_stack.pop()
        command.redo()
        self.undo_stack.append(command)
```

1. 执行新操作 (`execute`)：
    - 调用 `command.execute()`。
    - 将该 `command` 压入 **Undo栈**。
    - **清空 Redo栈**（因为一旦在撤销后执行了新操作，原本前方的重做历史就失效了）。

2. 点击撤销 (`undo`)：
    - 从 **Undo栈** 弹出（Pop）最近的一个命令。
    - 调用该命令的 `command.undo()`。
    - 将该命令压入 **Redo栈**。

3. 点击重做 (`redo`)：
    - 从 **Redo栈** 弹出最近的一个命令。
    - 调用该命令的 `command.redo()`。
    - 将该命令重新压入 **Undo栈**。 
