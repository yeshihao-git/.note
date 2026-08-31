---
tags:
  - git
---
# git 分支管理规范（简化版）

**使用流程**：
项目仓库中需要有测试用文件夹tests，并修改vitest.config.ts
组织中设置 scoped workflow 为组织中的工作流仓库
工作流仓库设置 关闭启用工作流

设置协作者
本地仓库第一次push到远程main后，再设置分支保护，管理员需遵守分支保护规则
仓库设置中设置分支保护main无法直接push和强制push，只能merge，merge需要指定一个人审批，若unit-test工作流未通过或审批未通过，则无法merge，merge后需要删除feature分支

# git 分支管理规范
## 分支结构

参考 git flow 工作流程：

![[Pasted image 20260727171055.png|547]]

### 长期分支

```
main
    |
    | 禁止push/force push
    | PR合并
    | 打tag，规则：v主版本.次版本.修订版本


develop
    |
    | 禁止push/force push
    | PR合并
```

### 临时分支

```
feature/*
    |
    | 从develop创建
    | 开发完成PR到develop
    | 
    删除


release/*
    |
    | 从develop创建
    | 发布时冻结develop
    | 只能bug修复
    |
    +--> PR到main，main再PR到develop
    |
    删除


hotfix/*
    |
    | 从main创建
    |
    +--> PR到main
    |
    +--> PR到develop
    |
    删除
```

## 合并策略与代码审查

| 来源        | 目标        | 策略               | 是否需要code review | 是否删除 | 原因       |
| --------- | --------- | ---------------- | --------------- | ---- | -------- |
| feature/* | develop   | **Squash Merge** | 必须              | 是    | 清理开发过程   |
| develop   | release/* | 创建分支             | 不需要             | 否    | 发布冻结     |
| release/* | main      | **Merge Commit** | 必须              | 是    | 保留发布历史   |
| main      | develop   | **Merge Commit** | 必须              | 否    | 回流bug修复  |
| hotfix/*  | main      | **Merge Commit** | 必须              | 否    | 保留线上修复历史 |
| hotfix/*  | develop   | **Merge Commit** | 必须              | 否    | 同步修复     |

## workflow 触发时机与执行内容（简化版）

| 分支/事件                   | Trigger      | Workflow | 动作               | 备注               |
| ----------------------- | ------------ | -------- | ---------------- | ---------------- |
| push feature            | push         | CI       | lint、test、build  |                  |
| merge feature → develop | pull_request | CI       | lint、test、build  | 平台配置：删除feature分支 |
| push release            | push         | CD-Test  | build + 部署test环境 |                  |
|                         |              |          |                  |                  |
| merge release → main    | pull_request | CD-Prod  | 生产部署 + 打tag      | 平台配置：删除release分支 |
| merge main → develop    | pull_request |          |                  |                  |
|                         |              |          |                  |                  |
| merge hotfix → main     | pull_request |          |                  |                  |
| merge hotfix → develop  | pull_request |          |                  | 平台配置：删除hotfix分支  |

## 文件夹结构与职责

|目录|作用|谁使用|
|---|---|---|
|repositories|源码|开发人员、CI|
|build|构建产物|CI/CD、回滚|
|deploy/test|测试环境|测试人员|
|deploy/prod|生产环境|用户|

**参考**：
- [上市公司Git分支管理规范](https://developer.aliyun.com/article/1366103?spm=5176.26934562.main.1.289312d4x3vVLY)
- [大厂git分支管理规范：gitflow规范指南](https://www.cnblogs.com/kevin-ying/p/14329768.html "发布于 2021-01-26 13:13")

# tmp
## CICD

```
开发提交代码
      │
      ▼
      CI（持续集成）
      │
      ├── 编译(Build)
      ├── Lint
      ├── UT 单元测试
      └── TA 自动化测试
      │
      ▼
通过
      │
      ▼
CD（持续部署）
      │
      ├── 部署测试环境
      ├── 部署预发布环境
      └── 部署生产环境
```

## git merge和rebase的选择

单人且临时的分支，使用rebase；其他情况使用merge


好的，我用 ASCII 图来直观展示 `merge` 和 `rebase` 的区别：

---

### 场景设定

假设你从 `main` 分支切出 `feature` 分支开发，期间 `main` 也有新提交：

```
初始状态：
A---B---C  main
    \
     D---E  feature

B 之后，main 有了新提交 F、G：
A---B---C---F---G  main
    \
     D---E        feature
```

现在要把 `feature` 的改动合并到 `main`。

---

### 方式一：Merge（合并）

```bash
git checkout main
git merge feature
```

**结果：**

```
A---B---C---F---G--------M  main
    \                    /
     D-----------------E  feature
                         ↑
                    merge commit
```

**特点：**
- 创建了一个新的 merge commit（`M`）
- 保留了完整的分支历史（有分叉）
- 真实记录了"什么时候合并的"
- 历史图看起来像"树枝分叉再汇合"

---

### 方式二：Rebase（变基）

```bash
git checkout feature
git rebase main
git checkout main
git merge feature
```

**第一步：** `feature` 的提交 `D`、`E` 被"移动"到 `G` 之后：

```
A---B---C---F---G  main
                \
                 D'---E'  feature
                 ↑
            D 和 E 被重写为 D' 和 E'
            （提交哈希变了）
```

**第二步：** `main` 快进到 `E'`：

```
A---B---C---F---G---D'---E'  main, feature
```

**特点：**
- 历史是一条直线，没有分叉
- 看起来像是"从头开始就在主分支上开发"
- 提交哈希被改写了（`D'` 和 `E'` 不同于原来的 `D`、`E`）
- 没有 merge commit
