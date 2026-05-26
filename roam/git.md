---
tags:
  - tools
---
# github 历史版本回退

```bash
git reset --hard <commit id>
git push -f
```

# .gitignore 中写入忽略 已跟踪文件 规则失效问题

在文件已被跟踪的情况下，在 .gitignore 中写入忽略规则是无效的，正确操作如下
1. 取消文件跟踪
```bash
git rm -r --cached <文件夹> # 若是文件不需要 -r
```
2. 在 .gitignore 中写入忽略规则
```bash
# .gitignore
<文件>
<文件夹>/
```
3. git commit

# 合并最近的两个 commit

1. 输入命令，接下来会打开一个文件
```bash
git rebase -i HEAD~2  # HEAD~2 表示：从最新 commit 往前数 2 个
```
2. 修改文件内容
```bash
# 最早的 commit 在上面，最新的在下面
pick a1b2c3d 第一个提交信息
pick e4f5g6h 第二个提交信息
```
3. 把第二个 commit 前面的 `pick` 改成 `s` 或 `squash`**
```bash
pick a1b2c3d 第一个提交信息
squash e4f5g6h 第二个提交信息 # squash = 把这个 commit 合并到上一个
```
4. 保存退出，接下来会打开另一个文件
5. 删除不需要的旧提交信息，保存退出
