---
epoch: 1785837165422
modelKey: "deepseek-v4-flash|deepseek"
topic: "Improve Gitea CICD Paragraph"



tags:
  - copilot-conversation
---

**user**: 这一段是使用gitea进行cicd的内容，帮我改的好一点
[Timestamp: 2026/08/04 17:52:45]

**ai**: 可以这样改写：

## 基于 Gitea 的 CI/CD 分支管理规范

### 1. 测试与工作流配置
- 项目仓库必须包含 `tests/` 测试目录，并在 `vitest.config.ts` 中完成测试相关配置。
- 在组织设置中，将 `scoped workflow` 指向组织中专门用于存放工作流文件的仓库。
- 在工作流仓库中，关闭“启用工作流”选项，使其仅作为组织级工作流来源，不再自行触发执行。

### 2. 协作者与初始推送
- 在仓库中配置协作者权限。
- 本地仓库首次推送 `main` 分支到远程后，再设置分支保护规则。
- 分支保护规则同样适用于管理员，管理员不得绕过规则直接推送。

### 3. main 分支保护规则
在仓库设置中为 `main` 分支启用分支保护：

- 禁止直接 `push` 和 `force push`，只能通过 Pull Request / Merge Request 合入。
- 合入请求必须至少经过 1 人审批。
- 合入前必须通过 `unit-test` 工作流；若测试未通过或审批未通过，则无法合并。
- 合并完成后，自动删除对应的 `feature` 分支。
[Timestamp: 2026/08/04 17:52:59]