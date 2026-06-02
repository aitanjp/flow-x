---
name: flow-health
description: >
  [project] 代码库健康巡检器。只产健康报告 + 改造建议清单，不直接改代码。
  支持工具路径（brooks-lint / jscpd / knip 等）和内置 AI 回退两种模式。
  Use when 用户说"健康检查/health/体检/技术债扫描/巡检/代码质量检查"，或周期性/里程碑前/接手陌生项目时。
paradigm: scout
trigger:
  - pattern: "健康检查|health|体检|技术债|巡检|代码质量|质量检查|债务扫描|debt|lint|sweep|audit"
    context: "用户想对代码库做健康度评估，不针对某个具体 change"
  - pattern: "每月检查|季度检查|版本发布前检查|接手项目|重构前"
    context: "周期性或里程碑前的健康巡检场景"
---

# flow-health — 代码库健康巡检

> **M 前缀表示 Maintenance / 横向命令**，不属于任何 change，不写 CHANGE.md / REQUIREMENT.md。直接产出健康报告。

## 角色

Maintenance Engineer。**只产健康报告 + 改造建议清单，不直接改代码**。

## 触发场景

- **周期性**：每月 / 每季度跑一次
- **里程碑前**：版本发布前 / 季度复盘 / 年终总结
- **接手陌生项目**：第一周必跑，了解整体健康度
- **重构决策**：决定要不要重构某模块前，先体检

## 输入

- 仓库根（不限单一 change）
- `.specs/CONTEXT.md`（含技术栈 + 团队偏好）
- `.specs/LESSONS.md`（已记录的失败教训）
- 最近 N 次的 `archive/<date>-<name>/` 归档（看历史趋势）
- 当前 git log 最近 30 天

## 你的职责

### 步骤 1 · 选模式

| 场景 | 推荐 | 输出深度 |
|---|---|---|
| 快速体检（5~10 分钟）| `/brooks-health` | 4 维仪表板 + 综合分 |
| 完整审计（30 分钟+）| `/brooks-sweep` | 6 维 + 6 测试维度 + 架构图 + 技术债 |
| 单维深挖 | `/brooks-audit` 或 `/brooks-debt` 或 `/brooks-test` | 单维度详细报告 |

**选择优先级**：
- 第一次跑 → `/brooks-sweep`（一次拿全貌）
- 周期性 → `/brooks-health`（轻量 · 看趋势）
- 拿到 health 报告后想深挖 → 对应单维命令

### 步骤 2 · 工具路径（首选）

尝试调用 `brooks-lint`（如项目已装）：
```bash
/brooks-sweep            # 全面扫
/brooks-health           # 健康仪表板
```

工具输出原样贴入报告。

### 步骤 2.5 · 冗余巡检（装/未装都要跑）

**为什么单独一步**：brooks-lint 的 R3 是**概念级**（同决策多处表达），本步补 4 个**字面级 + 死代码级**维度。

#### 2.5.1 语言检测

读 `.specs/CONTEXT.md`「技术栈」段 + 扫清单文件定主语言：

| 清单文件 | 语言 |
|---|---|
| `package.json` | JavaScript / TypeScript |
| `pyproject.toml` / `requirements.txt` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `pom.xml` / `build.gradle` | Java / Kotlin |
| `composer.json` | PHP |
| `Gemfile` | Ruby |

#### 2.5.2 调工具（首选）

| 维度 | 全语言 | TS/JS | Python | Go | Rust |
|---|---|---|---|---|---|
| 字面重复块 | `npx jscpd src/` | 同左 | 同左 | 同左 | 同左 |
| 未用导出 | — | `npx knip` / `npx ts-prune` | `vulture` | `deadcode` | `cargo udeps` |
| 未用依赖 | — | `npx depcheck` | `deptry` | `go mod tidy -v` | `cargo udeps` |
| 死代码 | — | ESLint `no-unreachable` | `vulture` | `staticcheck` | `cargo clippy` |

**执行策略**：
1. 优先 `jscpd`（全语言 · 5 分钟）：`npx jscpd src/ --min-lines 5 --min-tokens 50 --reporters json --output .specs/health/tmp/`
2. 再跑语言原生工具（按上表任选 1-2 个）
3. 工具未装 → 提示用户：`ℹ️ 需要跑《<cmd>》，是否授权自动安装？`（2 次拒绝 → 进 2.5.3 fallback）

#### 2.5.3 Fallback：AI grep 抓样

用户不肯装工具的退路：
- **重复块**：抽样 10 个最近改动的源文件，查 ≥ 5 行连续逻辑块在其他文件的出现
- **未用导出**：`grep "^export "` 拼符号后反向查引用数，0 次为候选
- **死代码**：只抽检明显的（`if false` / `return` 后的语句 / 永负分支）
- **未用依赖**：按清单文件逐条 grep 引用次数

在报告里标记：`⚠️ 内置 fallback 检测 · 精度低 · 漏报率高 · 建议装 jscpd / knip / vulture / staticcheck`

#### 2.5.4 严重度分级

| 分类 | 标准 |
|---|---|
| 🔴 **Critical** | ≥ 20 行重复块出现在 ≥ 3 处 · 被跨模块调用的公共导出无引用 · 死分支导致业务规则静默失效 |
| 🟡 **Major** | 5-19 行重复块 · 明显未使用的 public export · 清单文件顶层未用依赖 |
| 🟢 **Minor** | < 5 行重复 · 未使用的内部辅助函数 · 注释掉的死代码 · 孤立文档 |

#### 2.5.5 边界（避免误报）

- **测试文件不算未用**：`*.test.ts` / `*_test.go` 里的导出在源代码里无引用是正常的
- **公共 API 导出需人工确认**：lib / SDK 项目的 `index.ts` / `__init__.py` 导出供外部用。标 🟡候选 + 注"需确认是公共 API"
- **跨模块"重复块"常见假阳性**：模板化 CRUD / 单测 setup 常长得类似，语义不同 → 不算冷败，留 🟢提示
- **与 flow-review R3 协作**：本步是字面级 + 死代码级；flow-review R3 是概念级。**两者不互代替**

### 步骤 3 · 未装 brooks-lint（内置回退）

AI 自己按 6+6 维度过一遍主仓库的 `src/` / `lib/` / `app/`：

#### 3.1 生产代码 6 维抽样
随机抽 5 个最近改动频繁的模块（用 `git log --pretty=format: --name-only | sort | uniq -c | sort -rn | head -5`），每个按 6 维（R1~R6）打分。

#### 3.2 测试 6 维抽样
随机抽 5 个测试文件，按 T1~T6 打分。

#### 3.3 架构图
AI 用 grep + import 分析画简化 Mermaid 依赖图，标出循环依赖。

#### 3.4 综合分（自评）
每维度命中数 → 扣分（🔴 -10 / 🟡 -3 / 🟢 -1），从 100 起扣。**标注「内置回退评分，仅供参考，建议装 brooks-lint」**。

### 步骤 4 · 输出健康报告

写入 `.specs/health/<YYYY-MM-DD>-HEALTH.md`，结构见 `references/HEALTH.md` 模板。

### 步骤 5 · 反哺到 flow-x 工件

- 🔴 Critical → 自动开新 CHANGE（编号如 `health-fix-2026-q1`），进入 `flow-change`
- 🟡 Scheduled → 追加到 `.specs/CONTEXT.md`「技术债」段
- 🟢 Monitored → 追加到 `.specs/LESSONS.md`（用 `观察` 类型 entry）
- **冗余巡检特殊处理**：
  - 🔴 未用导出 / 死分支 → 合并到 health-fix CHANGE（不单开）
  - 🟡 重复块 → 记 CONTEXT.md「技术债」段，标"待抽公共函数"
  - 🟡 未用依赖 → **写到 CONTEXT.md「禁动清单」段**（标"下次清理窗口移除"），避免 AI 再次为已被移除的库写代码
  - 🟢 关联 `flow-evolve` 下次同步时**主动提醒**

## 约束

- **不直接改代码**（与 flow-review 同约束）
- **不开多个 fix CHANGE**：所有 Critical 合并成 1 个 health-fix change
- **基线对比强制**：第二次跑起，必须与上次报告对比

## 自检

- [ ] 选了正确的模式（首次 sweep / 周期 health / 单维深挖）
- [ ] 装了 brooks-lint 优先用工具输出，未装走内置回退并明确标注
- [ ] **步骤 2.5 冗余巡检已跑**：jscpd + 语言原生工具（装了）或 fallback grep（未装 · 已标 ⚠️）
- [ ] 综合分有「与上次对比」段
- [ ] Critical 项已生成 health-fix CHANGE 提案
- [ ] 未用依赖已写进 CONTEXT.md「禁动清单」
- [ ] 报告归入 `.specs/health/` 目录

## 触发下一步

- 有 🔴 Critical → 用户选「现在修」 → `flow-change` 开 health-fix change
- 全部 🟡/🟢 → 报告归档，提示用户「下次巡检建议日期：1 个月后」
