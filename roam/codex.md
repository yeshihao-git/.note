---
tags:
  - ai
  - vibe-coding
---
# codex
## AGENTS.md

**what**：
规则，约束项目

**how**：
可以在全局级或项目级加入
```
1. 不保留向后兼容。过时的直接删，别加兼容层、别写migration、别留fallback。（生产级项目不加）
2. 选能满足当前需求的最简单实现。不要预防性抽象，不要多此一举的配置层。
3. 系统分层长。先跑通一个最小的端到端版本，再往上加东西。绝不为了未完成的复杂度拆掉能跑的东西。
4. 组件保持模块化，关注点分离。
5. 优先用成熟的、有人维护的库。没有明确理由别自己重写。
6. 先翻项目里已有的依赖能做什么，再考虑加新包或自己写。别上来就假设库里没有。
7. 架构决策往长了做。不接受"先这样以后再换"的临时方案。
8. 先看成熟产品怎么解决同一个问题，用已验证的模式，别从零发明。
```

## 使用 ponytail skill

**what**：ponytail skill
极简代码。写代码少写、少依赖、少抽象，够用就停

**how**：
安装 ponytail skill
```powershell
npm install -g @openai/codex # 安装 codex cli
codex plugin marketplace add DietrichGebert/ponytail
codex plugin add ponytail@ponytail
```

codex 中使用
```codex
/ponytail lite   // 做正常方案，但提醒更懒的替代方案
/ponytail full   // 默认，尽量最小可用
/ponytail ultra  // 极简，能不写就不写
```
