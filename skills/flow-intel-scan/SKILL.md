---
name: flow-intel-scan
description: Brownfield 项目入场扫描器。自动探测项目技术栈、框架、既有抽象、命名约定和数据库 schema，生成/更新 `.specs/CONTEXT.md` 作为后续所有 change 的共享上下文。优先使用 GitNexus MCP 进行语义级扫描（执行流、模块社区、符号关系），GitNexus 不可用时回退到 grep/find。Use when 用户说"扫描代码/scan/intel/入场扫描/给项目体检"，或新项目首次使用 flow-x 且没有 AI 上下文文档时。
paradigm: scout
---

# flow-intel-scan — 入场扫描

## Goal

在已存在的项目（brownfield）里第一次用 flow-x 时，自动扫描代码库 → 自动填 `CONTEXT.md` → 后续所有 change 都受益。

不是周期命令，跑过一次就行；项目结构有大变化（重构 / 框架升级 / 新增模块）时再跑。

## GitNexus MCP 集成策略

**优先级**：GitNexus MCP（语义级） > grep/find（字面级）

| 扫描目标 | GitNexus MCP 工具 | 回退 grep 命令 |
|---|---|---|
| 模块/功能分区 | `cypher`（Community 节点 + heuristicLabel） | `ls src/` + `find src -type d` |
| 执行流/业务流程 | `query`（搜索关键概念） | `grep -rn "^import"` |
| 既有抽象层 | `query`（搜索 HTTP client / DAO / 状态管理等） | `grep -rn "axios\|fetch\|Repository"` |
| API 路由 | `route_map`（完整路由映射） | `grep -rn "@Route\|@Controller"` |
| 符号关系 | `context`（360度视图） | `grep -rn "class.*extends"` |
| 依赖方向 | `impact`（upstream blast radius） | `grep -rn "^import"` |

**可用性检测**（步骤 0 之后必跑）：
1. 调用 `gitnexus_impact` 传一个已知符号（如项目入口类），如果返回有效结果 → GitNexus 可用
2. 如果返回错误"index not found"或空结果 → GitNexus 不可用，全部回退到 grep
3. 如果返回"index stale"警告 → 提示用户：`npx gitnexus analyze` 刷新索引后再继续，或选择回退模式

**GitNexus 可用时**：步骤 1 全部用 GitNexus MCP 工具，grep 仅做补充验证。
**GitNexus 不可用时**：步骤 1 全部用原 grep/find 路径（已在步骤 1.1~1.6 中标注）。

## Workflow

### 0. 既有文档探测（必跑 · 在任何扫描前）

**目的**：很多项目已经有 AI 可读的开发约定文档。不要直接覆盖或忽略，先问用户。

探测下面这些标准文档（按优先级）：
- `CONTEXT.md`（仓库根 / `.specs/`）— flow-x
- `AGENTS.md`（仓库根）— OpenAI Codex
- `CLAUDE.md`（仓库根 / `.claude/`）— Anthropic
- `.cursor/rules/*.md` — Cursor
- `.windsurf/rules/*.md` — Windsurf
- `.github/copilot-instructions.md` — Copilot
- `.clinerules` — Cline
- `ARCHITECTURE.md`（仓库根 / `docs/`）— 项目自定义
- `CONTRIBUTING.md`（仓库根）— 项目自定义
- `README.md`（仓库根）— 通用保底

**三种分支**：

**分支 A** · 找到至少一个标准 AI 上下文文档 → 反问用户：
- 1. 综合+扫描（推荐）：读所有现有文档 + 跑入场扫描，合并生成 CONTEXT.md
- 2. 以现有文档为准：选定一个作为基础 → 提取关键信息到 CONTEXT.md
- 3. 忽略现有文档，重新扫描生成（警告：可能与现有约定冲突）
- 4. 不生成 `.specs/CONTEXT.md`，改用指定文档作为 AI 遵守依据

**分支 B** · 找到非标准但项目级文档（README / ARCHITECTURE / CONTRIBUTING）→ 反问用户：
- 1. 把它们当作扫描补充输入，生成完整 CONTEXT.md（推荐）
- 2. 跳过这些，仅按代码扫描
- 3. 用户指定其中一个作为开发遵守依据

**分支 C** · 一个文档都没有 → 反问用户（必须确认）：
- 1. 现在跑入场扫描，纯从代码生成 CONTEXT.md（~15-30k tokens，仅首次）
- 2. 手动指定项目里某个文档作为开发遵守依据
- 3. 跳过 intel-scan，直接进 0-change（不推荐 · AI 会"盲飞"）

无论选哪个分支，扫描总结里必须贴出：
- 探测命中文档列表
- 用户选择（选项编号 + 简述）
- 决策（生成 CONTEXT.md / 以现有为准 / 跳过）
- 引用源（如有）

### 0.5. GitNexus MCP 可用性检测（必跑 · 在步骤 1 前）

尝试调用 `gitnexus_impact`（target 取项目入口文件名或主类名，direction="upstream"）：
- **返回有效结果** → GitNexus 可用，记录 `gitnexus_available: true`，后续步骤 1 优先使用 GitNexus MCP
- **返回错误或空结果** → GitNexus 不可用，记录 `gitnexus_available: false`，全部回退到 grep
- **返回 stale 索引警告** → 反问用户：
  ```
  GitNexus 索引已过期（最后索引日期：<date>）。
  选项：
  1. 先跑 `npx gitnexus analyze` 刷新索引，再继续扫描（推荐 · 语义级精度更高）
  2. 跳过 GitNexus，用 grep/find 扫描（精度低但不需要等待）
  ```

在扫描总结里声明 GitNexus 使用状态。

### 1. 探测项目元信息

按下面顺序探测，记录每条发现（找不到也要记"未发现"）。

**GitNexus 可用时，优先使用 MCP 工具；不可用时使用 grep 命令。**

#### 1.1 包管理与运行时

| 信号文件 | 提取信息 |
|---|---|
| `package.json` | name / engines / 主要依赖 / scripts |
| `pyproject.toml` / `requirements.txt` / `Pipfile` | Python 版本 / 主要依赖 |
| `Cargo.toml` | Rust edition / 依赖 |
| `go.mod` | Go 版本 / 主要依赖 |
| `pom.xml` / `build.gradle` | Java 版本 / Spring / Maven |
| `composer.json` | PHP 版本 / Laravel / Symfony |
| `Gemfile` | Ruby / Rails 版本 |

#### 1.2 框架检测

**GitNexus 路径**：调用 `gitnexus_query` 搜索框架关键词（如 "Spring Boot controller"、"React component"），从返回的 process_symbols 中提取框架特征。

**grep 回退**：探测 React / Vue / Svelte / Next / Nuxt / NestJS / FastAPI / Django / Spring Boot / Express 等。

#### 1.3 关键约定

- 命名约定：`ls src/` 看目录命名风格
- import 风格：`grep "^import"` 看 alias 用法
- 测试框架：jest / vitest / pytest 等
- Lint / Format：eslint / prettier / ruff 等
- CI 平台：GitHub Actions / GitLab CI 等

#### 1.4 既有抽象层（关键 · 防重复实现）

**GitNexus 路径（优先）**：
对每类公共工具，调用 `gitnexus_query` 搜索对应概念：
- HTTP client：`gitnexus_query(query="HTTP request client", goal="find existing HTTP abstraction")`
- 数据库访问：`gitnexus_query(query="database access repository DAO", goal="find existing data layer abstraction")`
- 状态管理：`gitnexus_query(query="state management store", goal="find existing state abstraction")`
- 错误处理：`gitnexus_query(query="error handling exception", goal="find existing error abstraction")`

返回结果含 `process_symbols`（文件位置 + 模块归属），直接写入 CONTEXT.md「既有抽象索引」段。

**grep 回退**：
- HTTP client：`grep -rn "axios\|fetch\|httpClient\|apiClient" src/`
- 数据库访问：`grep -rn "Repository\|DAO\|prisma\|sequelize" src/`
- 状态管理：`grep -rn "createStore\|useStore\|atom" src/`
- 工具函数：`find src -path '*utils*' -o -path '*helpers*'`
- 自定义 hooks（前端）：`find src -name 'use*.ts*'`
- 中间件：`find src -path '*middleware*'`
- 错误处理：`grep -rn "class.*Error\|errorHandler\|ErrorBoundary" src/`

#### 1.5 数据库 schema

探测 Prisma / Alembic / Knex / Flyway / Liquibase / 裸 SQL 等工具及迁移目录。

#### 1.6 API 路由映射

**GitNexus 路径（优先）**：调用 `gitnexus_route_map` 获取完整路由映射（含 handler 文件、middleware wrapper 链、消费者）。输出直接写入 CONTEXT.md「跨模块契约」段。

**grep 回退**：`grep -rn "@Route\|@Controller\|@GetMapping\|@PostMapping" src/`

#### 1.7 模块/功能分区

**GitNexus 路径（优先）**：调用 `gitnexus_cypher` 查询社区/模块：
```
MATCH (c:Community) RETURN c.heuristicLabel, c.symbolCount, c.keywords, c.description
```
每个 Community 即一个功能分区，直接写入 CONTEXT.md「模块清单」段。

**grep 回退**：`ls src/` + `find src -type d -maxdepth 3`

### 2. 生成 CONTEXT.md

用 `references/CONTEXT.md` 模板，填入第 1 步的发现。每个字段都要有具体证据（文件路径 + 行号）。

**GitNexus 来源的证据格式**：`gitnexus://<tool>/<query> → <symbol>@<file>:<line>`
**grep 来源的证据格式**：`<file>:<line>`

**禁止**填空话："使用了一些工具" / "标准结构"——都要换成具体的证据级引用。

### 3. 写 STATE.md

```
last_intel_scan: <YYYY-MM-DD>
context_file: .specs/CONTEXT.md
detected_stack: <主要技术栈一句话总结>
gitnexus_available: <true/false>
gitnexus_last_index: <日期，如可用>
```

### 4. 给用户一份扫描总结

```
## 入场扫描完成

**项目类型**：<前端 / 后端 / 全栈>
**主要技术栈**：<Next.js 14 + Prisma + Postgres>
**GitNexus MCP**：<可用 / 不可用 · 回退到 grep>
**关键发现**：
  -  HTTP 客户端：`src/lib/api-client.ts:1`（统一封装 axios）
  -  工具函数：`src/utils/`（date / string / number 已有）
  -  自定义 hooks：`src/hooks/`（22 个）
  -  Schema 工具：未检测到（裸 SQL 或还没用迁移工具）
  -  测试覆盖：仅 `auth/` 目录有测试，其他模块裸奔
  -  模块分区：<GitNexus Community 列表 / grep 目录列表>

**对后续 change 的影响**：
  - 4-dev 阶段必须沿用既有 ApiClient（不要直接 fetch）
  - 涉及 schema 改动 → 建议引入 Prisma 或 Knex（4-dev 1.7 会拦）
  - 新增模块的 hooks 命名要参考现有 22 个的风格
  - **GitNexus 可用时**：后续 flow-dev / flow-architect / flow-review 将自动使用 GitNexus MCP

`.specs/CONTEXT.md` 已生成。后续运行 0-change 时 AI 会自动加载。
```

## Constraints

- **不动业务代码**：本工作流仅写 `.specs/` 内文件，禁动 `src/`
- **未经用户同意没自动开始扫描**：分支 C 必须等用户回复 1/2/3
- **每个字段都要有证据**：禁止"使用了一些工具"这类空话
- **CONTEXT.md 长度建议 ≤ 300 行**：超出时把陈旧条目归档到 `.specs/archive/CONTEXT-history.md`
- **GitNexus 优先但非强制**：GitNexus 可用时优先使用，不可用时回退到 grep。禁止在 GitNexus 不可用时阻塞扫描
- **GitNexus 索引过期必须提示**：不允许用过期索引产出结果而不告知用户

## Validation

- [ ] 步骤 0 既有文档探测做了：找到的标准 AI 上下文文档已列出，用户已选定分支
- [ ] 步骤 0.5 GitNexus 可用性检测做了：结果已记录到 STATE.md
- [ ] 未经用户同意没自动开始扫描（分支 C 必须等确认）
- [ ] 用户选 4「不生成 CONTEXT」时已写 STATE.md 的 `ai_context_doc` 并跳过剩余步骤
- [ ] 1.1~1.7 各段都有扫描输出（GitNexus MCP 或 grep），不靠猜
- [ ] CONTEXT.md 的每个字段都有文件路径 / 行号引用（或 GitNexus 来源标记）
- [ ] CONTEXT.md 顶部「源文档」段列出引用的 AGENTS / CLAUDE 等（如适用）
- [ ] STATE.md 的 `last_intel_scan` 和 `gitnexus_available` 已更新
- [ ] 扫描总结贴出关键发现 + 对后续 change 的影响 + GitNexus 使用状态

## Resources

- `references/CONTEXT.md` — 产出模板（项目共享上下文标准格式）
- GitNexus MCP 工具：`gitnexus_query` / `gitnexus_impact` / `gitnexus_context` / `gitnexus_cypher` / `gitnexus_route_map`
