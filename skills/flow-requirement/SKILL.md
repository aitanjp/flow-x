---
name: flow-requirement
description: 需求分析师。把 CHANGE.md 的粗粒度想法通过反问澄清为可执行需求 REQUIREMENT.md，含用户故事、Given/When/Then AC、v1/v2/out 范围切分、非功能性需求，并更新域语言到 CONTEXT.md。Use when 用户确认 CHANGE.md 后需要进入需求分析阶段，或已有 change 需要补充/修改需求时。
---

# flow-requirement — 需求分析

## Goal

把 CHANGE.md 的粗粒度想法通过反问澄清为可执行需求，提取域语言到 CONTEXT.md。

## Workflow

### 1. 写需求

使用 `references/REQUIREMENT.md` 模板填写：

- **用户故事**：以「作为<角色>，我想<动作>，以便<价值>」表达
- **验收准则（AC）**：每条用 Given/When/Then 结构，必须可被一条命令或一次手动操作验证
- **范围切分**：v1（本次必做）/ v2（下次再说）/ out（永远不做）
- **非功能性**：性能、可访问性、安全、兼容性等显式列出，没有就写"无"

### 2. 提取域语言（关键步骤）

在 `references/CONTEXT.md`（项目级）里**追加**或更新：
- **术语表**：本次引入的新名词，每个一句话定义
- **已锁决策**：本次确定的偏好
- **默认行为**：留给 AI 的可信默认值

> 域语言是 token 优化的基石。"主题切换的级联触发" 比展开描述短得多，但要先在 CONTEXT.md 里定义清楚。

### 3. 反问

任何不能被一句话验证的 AC，必须停下来反问。例：
- "界面要好看" → 反问："好看的标准是什么？是否对照某个设计稿？"
- "Lighthouse Performance >= 90" → 直接可验证

## Constraints

- 不允许写"如何实现"（那是 DESIGN 的事）
- AC 必须能被验证；不可验证的 AC 视为不合格
- 范围排除（v2 / out）至少各 1 条，否则说明范围切分还不够

## Validation

- [ ] 每条 AC 都有 Given/When/Then 结构
- [ ] 每条 AC 都能用一条命令或一次操作验证
- [ ] CONTEXT.md 至少新增 1 条术语或决策
- [ ] v1 / v2 / out 三类都有内容
- [ ] 非功能性需求显式列出（含"无"也要写）

## Resources

- `references/REQUIREMENT.md` — 产出模板
- `references/CONTEXT.md` — 域语言/术语表模板（追加模式）
