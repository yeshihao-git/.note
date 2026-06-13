---
tags:
  - cpp
  - 工具
---
# gdb

**how**：常用操作
```bash
# 启动gdb
gdb <二进制文件> [core]   # 启动 gdb 并加载程序 [coredump]
gdb -p <PID>              # 启动 gdb 并附加到正在运行的进程

# 查看代码
l                   # 查看当前行附加代码
l <函数名>          # 查看函数代码

# 打断点
b <行号> [if <exp>]                 # 行断点 [条件断点]
b [<文件名.cpp>]:[<类名>]:<函数名>  # 函数断点
b <文件名.cpp>:<行号>               # 头文件打断点（需禁用内联优化）

i b                                 # 查看断点
delete <断点编号>                   # 删除断点

# 运行程序
r                    # 运行程序
r <arg1> <arg2>      # 带参数运行程序

# 程序执行控制
n                    # 单步执行（不进入函数）
s                    # 单步执行（进入函数）
finish               # 执行完当前函数并返回
u <行号>             # 运行到某一行
c                    # 继续执行到下一个断点

# 调用栈分析
bt                   # 查看调用栈
f <栈帧号>           # 切换到执行栈帧

# 打印
p <变量名> [=<新值>]    # 打印变量值 [修改变量值]。打印string类型需要调用operator，eg：p name.operator=("hello") -> 将name赋值为 "hello"
ptype <变量名>          # 打印变量类型
watch <var> [if <exp>]  # 监视变量（值写入时暂停）
rwatch <var>            # 监视变量（值读取时暂停）
awatch <var>            # 监视变量（值写入、读取时暂停）
p <函数名>              # 执行函数，打印结果

# 多线程调试
i thread                 # 查看所有线程
thread <线程号>          # 切换到指定线程
show scheduler-locking   # 查看当前调度器锁（程序启动后才能设置成功）
set scheduler-locking on # 只允许当前线程执行
                         # off ：所有线程自由运行
                         # step：单步执行某一线程，线程视角不切换，其他线程同样单步执行；直到遇到某一断点，切换线程视角
```

## gdb 调试 C++ 项目

1. 配置调试信息、优化级别： `gcc` 中增加 `-g` 、`-O0` 选项

2. 使用 `gdb` 进行调试
```bash
gdb <可执行文件/core dump文件> # 启动 gdb 并加载程序 / core dump
gdb -p <PID> # 启动 gdb 并附加到正在运行的进程
```

3. `gdb` 常用操作
```bash
l # 查看源码
bt # 查看调用栈
f # 切换栈帧
b # 打断点
r # 运行/重新运行
n # 单步执行（不进入函数）
s # 单步执行（进入函数）
f # 从函数返回
c # 继续
q # 退出 gdb
p # 打印/修改变量
```

**具体调试示例**：调试 `core dump`
```bash
# 1. 启动 gdb 并打开 core dump文件
gdb <core dump文件>
# 2. 查看调用栈信息
bt 
# 3. 查看崩溃点
# 4. 打印变量值
p <变量>
# 5. 修改后重新测试
r
# 6. 没问题后退出 gdb
q
```

> **core dump**：
> `core dump` 文件为程序崩溃时的内存快照，用于分析程序崩溃原因

## 调试技巧：禁止下载调试信息

1. .gdbinit文件 是gdb的配置文件，通过 gdb --help 查看所在位置
2. 在 .gdbinit文件 写入 set debuginfod enabled off

**what**：debuginfod
一个HTTP客户端，用于下载丢失的调试信息

## 调试技巧：打印类中数据

1. 类中写一个print()函数
2. call print()
3. 强制刷新缓冲区（注意：cpp 需要强制刷新缓冲区才能看到结果）。以下两种方法：
	1. call (void)fflush(stdout)
	2. print() 中使用 cout << std::flush

##  调试技巧：调试cpp和python混合代码

```bash
gdb python -ex 'run xxx.py'
```

# core dump

**what**：
（核心转储文件）程序终止时的内存状态

**how**：
```bash
# linux设置程序崩溃时生成 core dump（linux 默认不生成 core dump）
ulimit -c unlimited   # 设置允许生成 core dump 文件
ulimit -c             # 查看值是否为 unlimited
ulimit -a             # 查看所有文件大小限制

# systemd中查看core dump与debug
coredumpctl list  # 查看 core dump
coredumpctl debug # 对 core dump 进行debug

# core dump存储位置
cat /proc/sys/kernel/core_pattern # 查看命名模板
# 若输出为 |/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h，说明 core dump 由 systemd 管理
# systemd管理的core dump存储位置：/var/lib/systemd/coredump
```

**what**：coredumpctl
systemd 中的工具；用于查看、管理 core dump文件
