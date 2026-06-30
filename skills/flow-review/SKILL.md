---
name: flow-review
description: 三轮审查官。只产报告+修复任务，不直接改代码。第一轮 spec 合规（AC 覆盖/范围蔓延），第二轮代码质量（6 维衰退风险诊断+书本引用），第三轮 UI 视觉（design tokens 一致性+anti-patterns 扫描+无障碍快检）。优先使用 GitNexus MCP 检测架构依赖和循环依赖。后端/lib 项目跳过第三轮。Use when 用户确认 TEST.md 后需要进入审查阶段，或需要 Code Review / 安全审计 / UI 视觉 audit 时。
---

# flow-review — 三轮审查

## Goal

作为 Reviewer，只产报告 + 修复任务，不直接改代码。

## Workflow

使用 `references/REVIEW.md` 模板分三轮审查（后端/lib 项目跳过第三轮）。

### 第一轮 · Spec 合规审查

逐条对照 REQUIREMENT.md 的 AC：
- [ ] 每条 AC 是否被实现
- [ ] 每条 AC 是否被测试覆盖（链接到 TEST.md）
- [ ] 是否引入了 `out of scope` 里明令排除的内容
- [ ] 是否新增了 REQUIREMENT.md 里没有的功能（范围蔓延）
- [ ] 是否触动了 DESIGN.md 之外的架构

### 第二轮 · 代码质量审查（书本驱动 6 维衰退风险）

#### 2.0 TEST.md 5 轮金字塔完整性

打开 TEST.md，检查"本次测试范围声明"段：
- 5 轮状态都明确（无未填）
- 跳过的轮次都有理由
- 第 1 轮每条 AC 有覆盖
- 第 2~5 轮按需达标

任意一项不达 → 标 🔴 Critical，先回 5-test 补完。

#### 2.1 代码质量诊断 · 6 维衰退风险

| 编号 | 衰退风险 | 诊断问题 | 主要源头 |
|---|---|---|---|
| R1 | Cognitive Overload | 理解这段代码要多少心智？ | Code Complete / Refactoring / DDD |
| R2 | Change Propagation | 改一点会坏多少不相干的地方？ | Clean Architecture / Pragmatic |
| R3 | Knowledge Duplication | 同一个决定被表达在多处？ | Pragmatic / Refactoring / DDD |
| R4 | Accidental Complexity | 代码是否比问题本身更复杂？ | Brooks / Philosophy of SD |
| R5 | Dependency Disorder | 依赖流是否一致方向？ | Clean Architecture / SE@Google |
| R6 | Domain Model Distortion | 代码是否忠实反映业务领域？ | DDD / Refactoring |

路径 A（装了 brooks-lint）：`/brooks-review`，输出必须含 4 要素（Symptom / Source / Consequence / Remedy）+ 书本引用，原样贴入 REVIEW.md。

路径 B（未装）：AI 自己逐个维度诊断，输出同样 4 要素格式。每条都要标 R1~R6 编号、具体 `<file>:<line>`、至少一本书作为 Source。

#### 2.2 架构依赖检查（大型 change 触发）

触发条件：新增/重命名顶级模块 / 危险 import / 引入新中间件 / 跨 >= 5 模块重构。

**GitNexus 路径（优先）**：
1. 调用 `gitnexus_cypher` 查询循环依赖：
   ```
   MATCH (a:Class)-[:CodeRelation {type: 'IMPORTS'}]->(b:Class)-[:CodeRelation {type: 'IMPORTS'}]->(a) RETURN a.name, a.filePath, b.name, b.filePath
   ```
2. 调用 `gitnexus_impact({target: "新增模块入口类", direction: "upstream"})` 检测反向依赖（domain 依赖 controller 等）
3. 调用 `gitnexus_cypher` 查询跨边界依赖（frontend 直接 import backend 等）
4. 所有检测结果直接写入 REVIEW.md，含符号名 + 文件路径 + risk 评级

装了 brooks-lint → `/brooks-audit`，拿到 Mermaid 依赖图。重点核：
- 循环依赖
- 反向依赖（domain 依赖 controller）
- 跨边界依赖（frontend 直接 import backend）

**grep 回退**（未装 brooks-lint + GitNexus 不可用）：AI 自己画简化 Mermaid 依赖图。

### 第三轮 · UI 视觉审查（仅前端项目）

触发条件：本次 change 含 UI-DESIGN.md 或 diff 涉及 UI 文件。

#### 3.1 Design Tokens 一致性
- [ ] 实现里的颜色值全部来自 UI-DESIGN.md frontmatter
- [ ] 无硬编码 hex/字号/间距（命中即 🔴 Critical）
- [ ] 字体与 UI-DESIGN.md 声明一致

#### 3.2 Anti-Pattern 扫描
逐项对照 `references/ui-anti-patterns.md` 的"强制禁忌"段：
- [ ] 字体类 / 颜色类 / 阴影类 / 边框类 / 动效类 / 布局类 / 文案类 / 组件类

每条命中必须列出 `文件:行号`，标 🔴 Critical 并生成 fix 任务。

#### 3.3 视觉北极星一致性
回到 UI-DESIGN.md 第 1 节"美学北极星"：
**"如果只看实现的截图，看得出来这个产品的调性是 <UI-DESIGN 声明的那个> 吗？"**
不能 → 标 🟡 Major。

#### 3.4 无障碍快检
- [ ] 颜色对比 >= WCAG 2.1 AA（工具实测）
- [ ] 所有交互元素键盘可达
- [ ] 焦点环可见
- [ ] `prefers-reduced-motion` 响应正确
- [ ] 表单 label 显式关联
- [ ] 图片 alt 文本

### 第四轮 · 补充审查（可选 · 按触发条件跳）

#### 4.1 技术债评估
触发条件：里程碑/季度大版本/重构项目，或 CONTEXT.md「技术债」段 > 30 天未更新。
装了 brooks-lint → `/brooks-debt`。Critical 本次必修，Scheduled 记入 backlog，Monitored 记入 LESSONS。

#### 4.2 跨模型 spot-check
触发条件：涉及安全/认证 / 涉及并发/分布式 / 单一函数 > 80 行 / 测试覆盖率显著下降。
拿另一个模型跑同样的三轮审查，差异填入 REVIEW.md「跨模型分歧」章节。

### 严重度分级

- 🔴 **Critical**：必须修复（数据损坏、安全漏洞、AC 未实现）
- 🟡 **Major**：建议修复（明显设计问题、显著性能回归）
- 🟢 **Minor**：可选改进（命名、风格、小重构）

### 产出修复任务

对所有 Critical 和决定要修的 Major，**追加到 TASK.md** 末尾，编号 `T-FIX-XX`，触发回到 flow-dev。

## Constraints

- 禁止直接修改代码
- 所有 Critical 必须修复或显式「已知接受」并经人工确认，否则禁止进 INTEGRATION
- 不允许笼统结论（"代码写得不错"），每条结论必须有具体行号或文件引用

## Validation

- [ ] 三轮主审查都做了（后端/lib 只跳第三轮 UI）
- [ ] 二轮 6 维诊断输出含 4 要素 + 书本引用 + R1~R6 编号
- [ ] 装了 brooks-lint 优先用，输出原样贴入报告
- [ ] 第四轮按触发条件判完
- [ ] 每条发现都有严重度标签
- [ ] 每个 Critical 都已生成 fix 任务
- [ ] 报告里没有自己悄悄改过的代码

## Resources

- `references/REVIEW.md` — 产出模板
- `references/ui-anti-patterns.md` — 反 AI-slop 清单（前端项目第三轮用）
