# Flow-X

Flow-X 是一套面向 claude code、code X、Lingma等的**编程智能体工作流框架**，通过标准化、可协作的阶段性工作流，将软件开发生命周期（SDLC）中的需求、设计、开发、测试、审查、集成等环节转化为结构化、可追溯、可复现的 AI 驱动流程。

## 定位

Flow-X 不是代码生成器，而是一套**流程编排与质量控制体系**。它通过 14 个专业化 Skill 构成完整的工作流管道，确保每一次变更（Change）都经过需求澄清、技术设计、任务拆解、TDD 开发、五轮测试、三轮审查、集成验收的完整闭环，最终沉淀为可维护的项目级知识资产。

## 核心特性

- **阶段化流水线**：从 Change 提案到归档，每个阶段有明确的输入输出和准入门槛（Artifact Preflight Gate）
- **质量内建**：TDD（RED→GREEN→REFACTOR）、五轮测试金字塔、6 维代码衰退风险诊断、UI 反 AI-slop 扫描
- **可追溯性**：每个变更都有唯一 change-id，所有产物（REQUIREMENT/DESIGN/TASK/SUMMARY/REVIEW）按变更隔离归档
- **知识沉淀**：LESSONS.md 记录跨任务失败教训，ARCHITECTURE.md / CONTEXT.md 积累项目级决策与抽象索引
- **Token 预算管理**：自动估算变更规模并让用户选择执行模式（完整 / 极简 / 单点）
- **Brownfield 友好**：既有项目自动检测已有 AI 上下文文档（AGENTS.md / CLAUDE.md / .cursor/rules 等），对齐既有架构

## 工作流全景

```
用户意图
    │
    ├── 新事物 → 0-change（变更提案）
    │                ↓
    │           1-requirement（需求分析）
    │                ↓
    │           2-design（技术设计）
    │                ↓
    │           2a-ui-design（UI 设计，前端项目）
    │                ↓
    │           3-task（任务拆解）
    │                ↓
    │           4-dev（单任务开发 / TDD）
    │                ↓
    │           5-test（五轮测试）
    │                ↓
    │           6-review（三轮审查）
    │                ↓
    │           7-integration（集成验收 + 归档）
    │
    ├── 横向命令（不属于任何 change）
    │       ├── I-intel-scan    代码扫描 / 生成 CONTEXT.md
    │       ├── A-architect     项目级架构梳理
    │       ├── A-evolve        架构沉淀同步
    │       ├── M-health        代码库健康巡检
    │       └── L-restyle       视觉风格切换
    │
    └── 恢复 → 加载 STATE.md → 从中断处继续
```

## 项目结构

```
flow-x/
└── skills/
    ├── flow-go/              # 统一入口 Orchestrator — 解析意图、路由阶段、估算预算
    ├── flow-change/          # 变更提案生成器 — 澄清想法、生成 CHANGE.md
    ├── flow-requirement/     # 需求分析师 — 用户故事、AC、范围切分
    ├── flow-design/          # 技术设计师 — 技术选型、架构图、ADR、风险分析
    ├── flow-ui-design/       # UI 美学导演 — Design Tokens、组件规约、反 AI-slop
    ├── flow-task/            # 任务拆解规划师 — 原子任务、波次依赖、并行标记
    ├── flow-dev/             # 单任务开发执行器 — TDD、既有抽象 grep、破坏性变更协议
    ├── flow-test/            # 五轮测试金字塔 — 功能/性能/安全/兼容/可观测
    ├── flow-review/          # 三轮审查官 — Spec 合规 / 代码质量 / UI 视觉
    ├── flow-integration/     # 集成验证与归档 — UAT、失败诊断、LESSONS 提名、归档
    ├── flow-architect/       # 项目级架构梳理 — 模块图、ADR、跨模块契约
    ├── flow-evolve/          # 架构演进同步器 — 批量同步沉淀到 CONTEXT/ARCHITECTURE
    ├── flow-health/          # 代码库健康巡检 — 冗余/死代码/技术债扫描
    ├── flow-intel-scan/      # 代码扫描 — 生成/更新 CONTEXT.md
    ├── flow-restyle/         # 视觉风格切换 — 已有项目换调性
    └── ...
```

每个 Skill 目录包含：

- `SKILL.md` — Skill 指令文件（触发条件、工作流、约束、验证清单）
- `references/` — 模板、规则节选、决策框架等参考材料

## 各 Skill 职责速查

| Skill | 阶段 | 角色 | 核心产出 |
|---|---|---|---|
| `flow-go` | 全局 | Orchestrator | 路由声明、阶段切换、预算估算 |
| `flow-change` | 0 | Partner | `.specs/<id>/CHANGE.md` |
| `flow-requirement` | 1 | Partner | `.specs/<id>/REQUIREMENT.md` |
| `flow-design` | 2 | Architect | `.specs/<id>/DESIGN.md`、ADR |
| `flow-ui-design` | 2a | Architect | `.specs/<id>/UI-DESIGN.md` |
| `flow-task` | 3 | Navigator | `.specs/<id>/TASK.md`（含 XML 任务） |
| `flow-dev` | 4 | Operator | `*-SUMMARY.md`、代码提交 |
| `flow-test` | 5 | Operator | `.specs/<id>/TEST.md` |
| `flow-review` | 6 | Scout | `.specs/<id>/REVIEW.md`、fix 任务 |
| `flow-integration` | 7 | Operator | `archive/<date>-<id>/`、CHANGELOG |
| `flow-architect` | A | Architect | `.specs/ARCHITECTURE.md` |
| `flow-evolve` | A | Philosopher | `.specs/evolve/<date>-EVOLVE.md` |
| `flow-health` | M | Scout | `.specs/health/<date>-HEALTH.md` |
| `flow-intel-scan` | I | Navigator | `.specs/CONTEXT.md` |
| `flow-restyle` | L | Partner | `UI-DESIGN.md` v2 |

## 产物规范

Flow-X 在项目中使用 `.specs/` 目录管理所有变更级和项目级产物：

```
<repo-root>/
├── .specs/
│   ├── CONTEXT.md                 # 项目级：术语表、已锁决策、既有抽象索引
│   ├── ARCHITECTURE.md            # 项目级：模块图、ADR、跨模块契约（可选）
│   ├── LESSONS.md                 # 项目级：跨任务失败教训知识库
│   ├── CHANGELOG.md               # 项目级：变更历史
│   ├── <change-id>/
│   │   ├── CHANGE.md
│   │   ├── REQUIREMENT.md
│   │   ├── DESIGN.md
│   │   ├── UI-DESIGN.md           # 前端项目
│   │   ├── TASK.md
│   │   ├── T01-SUMMARY.md
│   │   ├── T02-SUMMARY.md
│   │   ├── TEST.md
│   │   ├── REVIEW.md
│   │   └── UAT.md
│   ├── archive/
│   │   └── <YYYY-MM-DD>-<change-id>/
│   ├── health/
│   │   └── <YYYY-MM-DD>-HEALTH.md
│   └── evolve/
│       └── <YYYY-MM-DD>-EVOLVE.md
└── STATE.md                       # 活跃 change、当前阶段、中断任务
```

## 使用方式

Flow-X 作为 Lingma 的 Skill 集合使用。将 `skills/` 下的各 Skill 目录配置到 Lingma 的 Skill 加载路径中，即可通过自然语言触发工作流。

**典型入口指令示例：**

- "我想做一个用户登录功能" → 触发 `flow-change`，启动完整流水线
- "执行 T03" → 触发 `flow-dev`，执行 TASK.md 中的 T03 任务
- "继续" / "恢复" → 加载 `STATE.md`，恢复中断的开发任务
- "审查代码" → 触发 `flow-review`，执行三轮审查
- "健康检查" → 触发 `flow-health`，扫描代码库技术债
- "同步架构" → 触发 `flow-evolve`，批量沉淀架构决策

## 设计原则

1. **人工在环（Human-in-the-loop）**：关键决策（技术选型、范围切分、ADR 确认）必须经用户确认，AI 不替用户做不可逆决定
2. **小步快跑**：每个开发任务控制在 2~10 分钟可完成的原子粒度，支持中断恢复
3. **失败即知识**：每次试错耗时 > 30 分钟或具有复用价值的失败，必须沉淀到 LESSONS.md
4. **只读不动**：审查、健康检查、架构梳理等 Scout/Architect 角色只产报告，不直接修改业务代码
5. **Token 效率**：通过 CONTEXT.md 的域语言和既有抽象索引，减少重复上下文消耗

## 约束红线

- `flow-dev`：verify 未通过禁止标记完成；破坏性变更必须走 grep 引用图 + 反问协议
- `flow-review`：禁止直接修改代码；所有 Critical 必须修复或经人工确认
- `flow-test`：测试用例从 AC 派生，不从实现派生；禁止通过删除/弱化测试来"修复"失败
- `flow-integration`：归档操作必须用户确认；UAT 失败自动重试不超过 3 轮
- `flow-go`：Preflight 失败必须回退；禁止要求用户提供 ID/路径/阶段名
