---
tags:
  - 工具
---
# git

**what**：
用于版本管理的命令行工具，包含四个重要的区域：
1. 工作区（Working Directory）
2. 索引（Index）
3. 仓库（HEAD）
4. 远程仓库

## github 历史版本回退

```bash
git reset --hard <commit id>
git push -f
```

## .gitignore 中写入忽略 已跟踪文件 规则失效问题

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

## git合并最近的两个 commit

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
3. 把第二个 commit 前面的 `pick` 改成 `s` 或 `squash`
```bash
pick a1b2c3d 第一个提交信息
squash e4f5g6h 第二个提交信息 # squash = 把这个 commit 合并到上一个
```
4. 保存退出，接下来会打开另一个文件
5. 删除不需要的旧提交信息，保存退出

## git 远程仓库commit覆盖本地commit

```bash
git fetch
git reset --hard origin/main
```

## git解决ssh: connect to host github.com port 22: Connection timed out 

1. 在 ~/.shh/config 中写入以下内容
```bash
Host github.com
 Hostname ssh.github.com
 Port 443
```
2. ssh -T git@github.com 测试是否成功

## git配置SSH key

```bash
ssh-keygen -t rsa -C "1299331829@qq.com" # 生成 SSH key 到 ~/.ssh
# 复制 id_rsa.pub 中的内容到 github/Settings/SSH and GPG keys
```


## git push密码认证失效

**问题复现**：
出现了要求输入 账号密码 的提示
```bash
root@autodl-container-92714881b1-6850edac:~/autodl-tmp/MIG-GT# git push --set-upstream origin matplotlib
Username for 'https://github.com': diablorrr
Password for 'https://diablorrr@github.com':
remote: Support for password authentication was removed on August 13, 2021.
remote: Please see https://docs.github.com/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls for information on currently recommended modes of authentication.
fatal: Authentication failed for 'https://github.com/diablorrr/MIG-GT.git/'
```
查看发现 推拉 走的 HTTPS方式
```bash
root@autodl-container-92714881b1-6850edac:~/autodl-tmp/MIG-GT# git remote -v
origin  https://github.com/diablorrr/MIG-GT.git (fetch)
origin  https://github.com/diablorrr/MIG-GT.git (push)
```

**原因**：
github 在 2021.8.13 移除了对账号密码认证的支持；因此无法使用帐号密码通过 HTTPS 推送代码

**解决**：
1. 生成 公钥密钥：见 git配置SSH key
2. 设置推拉方式为 SSH：
```bash
git remote set-url  origin git@github.com:diablorrr/MIG-GT.git
```

## git rebase修改任意commit的提交信息

```bash
git rebase -i --root
# 选择commit，更改 pick 为 reword，保存退出
# 修改commit，保存退出
```

## git恢复误删除的分支

```bash
git reflog
git checkout -b <lost-branch> <commit-id>
```

## git报错：The TLS connection was non-properly terminated 

**报错信息**：
```
fatal: unable to access 'https://github.com/CMU-Perceptual-Computing-Lab/openpose.git/': gnutls_handshake() failed: The TLS connection was non-properly terminated.
```

**解决**：
1. 权限不足，用 sudo：
```bash
sudo git clone ...
```
2. 取消代理：
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## git clone加速

**方法1** ：git走代理

**方法2** ：gitee镜像
```bash
# Gitee极速下载 中查看是否有自己需要的库
git clone https://gitee.com/mirrors/仓库名.git
```

**方法3** ：gitee中转
1. gitee右上角 => 从github/gitlab导入仓库
2. 复制github仓库地址

**方法4** ：嵌入gitclone.com
```bash
git clone https://gitclone.com/github.com/pzl/oa1.git # 嵌入后
git clone https://github.com/pzl/oa1.git              # 嵌入前
```

## git 规范
### git commit规范

```bash
<commit类型>[<commit涉及范围>]: <commit简短描述>

<body>

<footer>
```

<commit类型>如下：

|类型|说明|
|---|---|
|feat|新增 feature (新功能)|
|fix|修复 bug|
|docs|仅仅修改了文档|
|style|仅仅修改了空格、格式缩进、逗号等等，不改变代码逻辑|
|refactor|代码重构，没有 加新功能或者 修复 bug 的代码变动|
|perf|优化相关，比如提升性能、体验|
|test|增加 测试用例|
|chore|构建工具或辅助工具的变动|
|revert|回滚到上一个版本|
|merge|代码合并|
|sync|同步主线或分支的 bug|
### git branch规范

| 分类                      | 分支            | 作用                                                                |
| ----------------------- | ------------- | ----------------------------------------------------------------- |
| 长期分支                    | master        | (主分支) 上线版本， hotfix 分支从该分支创建                                       |
|                         | develop       | (开发分支) 相对稳定版本， feature 分支、bugfix 分支、release 分支都从该分支创建             |
| 短期分支<br><br>(完成功能开发后删除) | feature / 分支名 | (功能分支) 开发自测后，合并到 develop 分支，删除该分支                                 |
|                         | bugfix / 分支名  | (bug 修复分支) 用于修复不紧急的 bug，从 develop 创建，开发自测后，合并到 develop 后，删除该分支    |
|                         | release / 分支名 | (预发布分支) 从 develop 创建，发布到测试环境测试，发现 bug 修复后，合并到 master 和 develop，上线 |
|                         | hotfix / 分支名  | (紧急 bug 修复分支) 从 master 分支创建，用于紧急修复线上 bug，修复后，合并到 master 和 develop |

## git diff信息详解

```bash
1 diff --git a/test b/test           # 表示git正在比较两个文件
2 index a199eb7..1961728 100644      # git对象哈希值和文件权限：100普通文件，644权限644
3 --- a/test                         # 修改前文件
4 +++ b/test                         # 修改后文件
5 @@ -1,2 +1,2 @@                    # -1,2：旧文件第1行开始2行内容 +1,2：新文件第1行开始3行内容
6  happy1
7 -happy1                            # - 代表删除内容
8 +ysh                               # + 代表新增内容
```

## .gitignore语法

```bash
*.txt                          # 忽略某种类型的文件
*.[ab]                         # 通配符方式过滤：过滤所有以.a或.b扩展名的文件
target/                        # 忽略文件夹下的所有文件
```

| 符号  | 作用                 |
| --- | ------------------ |
| #   | 注释                 |
| /   | 目录                 |
| *   | 通配多个字符             |
| ?   | 通配单个字符             |
| []  | 单个字符匹配列表           |
| !   | 不忽略 (跟踪) 匹配到的文件或目录 |

## git配置文件

|配置文件|作用|查询方法|
|---|---|---|
|/etc/gitconfig|git 系统级配置|git config --system --list|
|~/.gitconfig|git 用户级配置|git config --global --list|
|<仓库路径>/.git/config|git 仓库级配置|git config --local --list|
|.gitignore|配置 git 忽略跟踪的文件 / 文件夹||
|.gitmodules|配置 git 各个子模块的仓库地址||
