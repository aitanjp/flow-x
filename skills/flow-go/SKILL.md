---
name: flow-go
description: flow-x 统一入口 Orchestrator。解析用户意图，自动匹配阶段，执行 Artifact Preflight Gate，估算 Token 预算，加载必要规则与工件，路由到对应子 skill 执行。Use when 用户说"开始做某事/继续/审查/测试/设计/换风格/健康检查/扫描代码/架构梳理"等任何与 flow-x 流程相关的意图，或需要自动管理 change 生命周期时。
---

# flow-go — flow-x 统一入口

## Goal

作为 flow-x 编程智能体蜂群的唯一入口，解析用户一句话意图，自动完成：
1. 读取项目状态（STATE.md）
2. 执行 Artifact Preflight Gate（检查上游工件完整性）
3. 解析用户意图并路由到正确阶段/子 skill
4. 估算 Token 预算并让用户选挡位
5. 自动准备（生成 change-id、加载规则与工件）
6. 显式声明执行计划后进入对应子 skill

## Workflow

### 第一步 · 读取项目状态

1. 尝试读仓库根 `STATE.md`。不存在 → 视为新项目
2. 关注字段：`活跃 Change` / `当前阶段` / `当前 Task` / `中断任务`
3. 存在 `中断任务` 非空 → **优先级最高**，直接走"恢复中断任务"分支

### 第二步 · Artifact Preflight Gate（强制）

路由到任何阶段前，检查上游工件是否齐全：

| 目标阶段 | 必须已有的上游工件 | 缺失时动作 |
|---|---|---|
| `0-change` | 无 | 直接进入 |
| `1-requirement` | `.specs/<id>/CHANGE.md` | 回 `0-change` |
| `2-design` | `CHANGE.md` + `REQUIREMENT.md` | 缺哪个回哪个 |
| `2a-ui-design` | `CHANGE.md` + `REQUIREMENT.md` + `DESIGN.md` | 回缺失阶段 |
| `3-task` | `REQUIREMENT.md` + `DESIGN.md`；前端还须有 `UI-DESIGN.md` | 回缺失阶段 |
| `4-dev` | 正式 `.specs/<id>/TASK.md` 中的当前 task，或用户显式临时最小 TASK | 反问：回 `3-task` 还是用户提供临时 TASK |
| `5-test` | `REQUIREMENT.md` + `DESIGN.md` + `TASK.md` + 各 `*-SUMMARY.md` | 回缺失阶段 |
| `6-review` | `REQUIREMENT.md` + `TASK.md` + `TEST.md` + 本次 diff | 回缺失阶段 |
| `7-integration` | `.specs/<id>/` 下本 change 全部应有产物 | 回缺失阶段 |

Preflight 失败时输出：
```
规则 R2.7 触发：目标阶段缺少 <工件>。本次先回到 <阶段> 补齐，不能直接继续。
```

### 第三步 · 解析用户意图，路由到阶段

**优先级判定**：
- 当前无活跃 change + 用户说"做/想/加/实现/设计 + X" → **优先路由到 `0-change`**（新事物）
- 当前有活跃 change + Artifact Preflight 通过 → 按以下表格匹配

**意图匹配表**（取最先命中）：

| 用户输入特征 | 路由到 | 备注 |
|---|---|---|
| `继续` / `接着上次` / `恢复` / `resume` | `4-dev` 入场恢复 | 加载 STATE.md 中断任务对应的 PROGRESS |
| `执行 T<NN>` / `跑 T<NN>` / `do T<NN>` | `4-dev` | task-id 从用户输入提取 |
| `审查` / `review` / `检查代码` / `code review` | `6-review` | |
| `测试` / `写测试` / `UAT` / `test` | `5-test` | |
| `上线` / `集成` / `验收` / `ship` / `归档` | `7-integration` | |
| `拆任务` / `plan tasks` / `分解` | `3-task` | |
| `设计` + `<已有需求>` / `架构` / `design` | `2-design` | 仅当 CHANGE + REQUIREMENT 已存在 |
| `选技术` / `选栈` / `tech stack` | `2-design` 步骤 0 | 只需技术栈选型 |
| `UI` / `视觉` / `美学` / `theme` / `design tokens` | `2a-ui-design` | 前端项目 |
| `换调性` / `改风格` / `restyle` / `重做视觉` | `L-restyle` | 已有项目换视觉 |
| `健康检查` / `health` / `体检` / `扫冗余` / `技术债扫描` | `M-health` | 代码库周期性巡检 |
| `扫描代码` / `scan` / `intel` / `入场扫描` | `I-intel-scan` | 生成/更新 CONTEXT.md |
| `同步架构` / `沉淀架构` / `evolve` / `架构演进` | `A-evolve` | 扫归档 DESIGN §9 批量同步 |
| `建立架构` / `架构梳理` / `architect` / `画架构图` | `A-architect` | 首次/重构建立 ARCHITECTURE.md |
| `需求` / `spec` / `requirement` | `1-requirement` | |
| 任何**新事物描述**（"做/想/加/实现/设计 + X"，且无活跃 change） | `0-change` | **自动生成 change-id** |
| 模糊不清 | 反问用户 | 新需求 / 继续上次 / 别的？ |

> **架构级变更二次拦截**：路由到 0-change / 2-design 时，必须先按 0.4 / 0₋ 做架构级预检。

### 第四步 · Token 预算估计

用户首轮路由后，AI 必须输出预算估算：

```
Token 预算估计：
   - 本次 change 规模：<small / medium / large>
   - 默认模式预估：~XXk - YYk tokens
   - 已选挡位：完整 / 极简 / 单点
是否继续？或换挡位？
   1. 完整（推荐 500+ 行 / 团队项目 / 长期维护）
   2. 极简（推荐 100~500 行 · 非 UI 项目可跳 2a / 跳第四轮 / 跳跨模型，省 ~20%）
   3. 单点（你只想跑某一阶段，告诉我哪一个）
   4. 不走 flow-x（< 50 行代码 / bugfix 直接修）
```

**成本影响因子**：
| 因子 | 影响 |
|---|---|
| 前端项目 | +20% |
| 涉及 schema 变更 | +5~10% |
| brooks-lint 已装 | +10% |
| 跨模型 spot-check | +30% |
| task 数 < 3 | -30% |
| task 数 > 10 | +50% |

**不必估预算的情况**：用户已显式指定模式 / 恢复中断任务 / 跑横向命令

### 第五步 · 老项目入场检测（brownfield 必跑）

**触发**：进入 0-change 之前必跑。

探测以下 AI 上下文文档（按优先级）：
- `CONTEXT.md`（仓库根 / `.specs/`）— flow-x 自己
- `AGENTS.md`（仓库根）— OpenAI Codex
- `CLAUDE.md`（仓库根 / `.claude/`）— Anthropic
- `.cursor/rules/*.md` — Cursor
- `.windsurf/rules/*.md` — Windsurf
- `.github/copilot-instructions.md` — Copilot
- `.clinerules` — Cline

**判决**：
- **A** · 找到标准 AI 文档 → 反问用户：综合+扫描 / 以现有为准 / 忽略重扫 / 不生成 CONTEXT
- **B** · 找到 README/ARCHITECTURE/CONTRIBUTING → 反问用户：当作补充输入 / 仅代码扫描 / 指定遵守依据
- **C** · 无任何文档 → 反问用户：跑扫描 / 手动指定 / 跳过（不推荐）
- **D** · 刚创新项目（无 package.json 等）→ 跳过，CONTEXT 在后续阶段逐步沉淀
- **E** · STATE 已设 `ai_context_doc` → 直接读它，不再询问
- **F** · CONTEXT.md 存在且 last_intel_scan 在 90 天内 → 直接读，跳过本步
- **G** · CONTEXT.md 存在但 > 90 天 → 读 + 提醒用户可重扫

### 第六步 · 自动准备

进入对应阶段前完成：
- **新 CHANGE**：自动生成 `change-id`（kebab-case，2~4 词），检查冲突自动加序号
- **目录**：自行 `mkdir -p .specs/<id>/`
- **规则加载**：读 `references/SYSTEM.md`（精简规则）和 `references/RULES.md`（硬规则 R1~R8）
- **外部扩展检测**：检查 brooks-lint / ui-ux-pro-max / impeccable 是否在路由声明中标明

### 加载工件策略

**语义约定**：
- `⚡︎ 全读`：进阶段首轮必须 read_file 整个文件（SPEC / TEMPLATE）
- `⚡︎ 查表`：只 grep 指定节或 read offset/limit，**禁止默认整读**（reference/*）
- `⚡︎ 按需`：首轮不读，需要时才 grep / 读

| 阶段 | 全读（SPEC） | 查表（REFERENCE） | 按需 |
|---|---|---|---|
| 0 / 1 | — | `ui-aesthetics.md` 只查「给 AI 在 0-change 阶段展示用的标准模板」一节（仅前端）| — |
| 2 | `CHANGE.md` + `REQUIREMENT.md` + `CONTEXT.md` + `ARCHITECTURE.md`（如存在）| `tech-stacks.md` 只查「适用矩阵」+ 过滤出的 5~6 张卡片 | ADR 阶段某项要深谈时再读 |
| 2a | `CHANGE.md` + `REQUIREMENT.md` + `DESIGN.md` `## 0` 段 + `CONTEXT.md` + `ui-anti-patterns.md`（75 行可全读）| `ui-aesthetics.md` 查「5 维度」+ 「给 AI 的模板」| uipro / impeccable 查询 |
| 3 | `REQUIREMENT.md` + `DESIGN.md` + `UI-DESIGN.md`（前端）+ `CONTEXT.md` | — | 任务模板查询 |
| 4 | `TASK.md`（只读当前 task 块）+ `DESIGN.md` `## 0` 段 + `UI-DESIGN.md`（UI 任务）+ `CONTEXT.md` + `LESSONS.md` | `ui-anti-patterns.md`（UI 任务 · 75 行）| — |
| 5 | `REQUIREMENT.md` + `DESIGN.md` `## 0` 段 + `TASK.md` + 各 `*-SUMMARY.md` | `test-pyramid.md` 只查「适用矩阵」+ 需要轮次 | — |
| 6 | `REQUIREMENT.md` + `DESIGN.md` + `TASK.md` + `TEST.md` + `git diff` | `ui-anti-patterns.md`（前端项目第三轮 · 75 行）| — |
| 7 | `.specs/<id>/` 全部产物 + `LESSONS.md` | — | — |

### 第七步 · 显式声明执行计划（必须）

进入实际工作前，输出**路由声明**：

```
路由：<阶段，例如 0-change>
Change-ID：<id>（已自动生成 / 已恢复活跃 change：<existing-id>）
已加载：
   - <file1>（全读，N 行）
   - <file2>（全读，N 行）
   - <reference-file>（仅查「某节」，line X-Y）
未加载：<本阶段不需但后面可能用到的 reference，明说何时才拉>
第一动作：<具体下一步>
```

### 第八步 · 执行对应子 skill

加载并执行路由到的子 skill 的 instructions。所有规则（`RULES.md` / `SYSTEM.md`）继续生效。

**子 skill 清单与调用条件**：

| 子 skill | Paradigm | 调用条件 | 产出 Artifact |
|---|---|---|---|
| `flow-intel-scan` | Navigator | 新项目首次使用 / 用户说"扫描代码" | `.specs/CONTEXT.md` |
| `flow-architect` | Architect | 首次建立 / 重大重构 / 画架构图 | `.specs/ARCHITECTURE.md` |
| `flow-change` | Partner | 无活跃 change + 新事物 | `.specs/<id>/CHANGE.md` |
| `flow-requirement` | Partner | CHANGE.md 已确认 | `.specs/<id>/REQUIREMENT.md` |
| `flow-design` | Architect | REQUIREMENT.md 已确认 | `.specs/<id>/DESIGN.md` |
| `flow-ui-design` | Architect | DESIGN.md 已确认 + 前端项目 | `.specs/<id>/UI-DESIGN.md` |
| `flow-task` | Navigator | DESIGN.md（+ UI-DESIGN.md）已确认 | `.specs/<id>/TASK.md` |
| `flow-dev` | Operator | TASK.md 已确认 + 用户说"执行 T<NN>" | `*-SUMMARY.md` / `*-PROGRESS.md` |
| `flow-test` | Operator | 所有 DEV 任务完成 | `.specs/<id>/TEST.md` |
| `flow-review` | Scout | TEST.md 已确认 | `.specs/<id>/REVIEW.md` |
| `flow-integration` | Operator | REVIEW.md 已通过 | `archive/<date>-<id>/` + CHANGELOG |
| `flow-restyle` | Partner | 已有项目换视觉调性 | `.specs/<id>/UI-DESIGN.md` v2 |
| `flow-health` | Scout | 周期性巡检 / 里程碑前 / 接手项目 | `.specs/health/<date>-HEALTH.md` |
| `flow-evolve` | Philosopher | 每月/每季度批量同步沉淀 | `.specs/evolve/<date>-EVOLVE.md` + patch |

**Artifact 传递协议**：
- 所有 change 级产物写入 `.specs/<change-id>/`
- 项目级产物（CONTEXT / ARCHITECTURE / LESSONS / health / evolve）写入 `.specs/`
- 归档产物写入 `.specs/archive/<YYYY-MM-DD>-<change-id>/`
- 状态文件 `STATE.md` 在仓库根，由 flow-go 和各子 skill 共同维护
- `PROGRESS.md` / `LESSONS.md` 作为跨 change 共享状态，位于 `.specs/`

## Decision Tree

```
[用户输入]
    │
    ├── 空输入 → 反问用户：新想法 / 继续上次 / 审查代码
    │
    ├── 含"继续/恢复/resume" → 加载 STATE.md → 提取中断任务 → 4-dev 入场恢复
    │
    └── 【活跃 change 检测】
            │
            ├── 有活跃 change + Preflight 通过 → 【意图匹配】按表格路由
            │
            └── 无活跃 change 或 Preflight 失败
                    │
                    ├── "做/想/加/实现/设计 + X" → 0-change（生成新ID）
                    ├── "扫描代码/入场扫描" → I-intel-scan
                    ├── "健康检查/体检" → M-health
                    ├── "建立架构/画架构图" → A-architect
                    ├── "同步架构/沉淀" → A-evolve
                    └── 其他 → 反问用户
```

## Constraints

- **Token 预算红线**：进入任何阶段的首轮消息，加载的 reference/* 总行数 ≤ **150 行**
- **禁止默认整读 reference**：`tech-stacks.md`(467行) / `ui-aesthetics.md`(350行) / `test-pyramid.md`(200行) 必须先 grep 标题再 read offset
- **ui-anti-patterns.md 唯一可整读**：仅 75 行
- **不得要求用户提供 ID/路径/阶段名**：这些 AI 自己决定
- **Preflight 失败必须回退**：禁止伪造"已满足"
- **新事物优先走 0-change**：即使有"设计/UI/测试"等关键词，无活跃 change 时仍先走 change

## Validation

产出路由声明前自检：
- [ ] 已读 STATE.md（如果存在）
- [ ] 已按表格匹配意图，没有跳过
- [ ] 新 CHANGE 已自动生成 ID 并展示
- [ ] **Token 预算**：本轮加载的 reference/* 总行数 ≤ 150
- [ ] **未越界**：没有读「查表」或「按需」列中的文件为全文
- [ ] 路由声明含「已加载 / 未加载 / 起止行」三要素
- [ ] 没有要求用户提供 ID / 路径 / 阶段名

## Resources

- `references/SYSTEM.md` — RULES + METHODOLOGY 精简版（永久注入用）
- `references/RULES.md` — R1~R8 完整硬规则
- `references/METHODOLOGY.md` — 方法论骨架（阶段定义、文件体系、7机制）
