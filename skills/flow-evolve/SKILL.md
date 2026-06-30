---
name: flow-evolve
description: >
  [project] 架构演进同步器。把已归档 change 的 DESIGN.md §9 架构沉淀
  同步到项目级文档（CONTEXT.md + ARCHITECTURE.md）。只读 §9，只 patch 文档，不动业务代码。
  GitNexus MCP 可用时用于检测代码与文档之间的偏差（新抽象未入索引、ADR 与实际代码不一致）。
  Use when 用户说"同步架构/整理沉淀/sediment/evolve/架构演进/同步 CONTEXT"，或每月/每季度批量同步时。
paradigm: philosopher
trigger:
  - pattern: "同步架构|整理沉淀|sediment|evolve|架构演进|同步 CONTEXT|同步文档|升级 CONTEXT|升级架构文档"
    context: "用户想把多个已完成的 change 的架构沉淀合并到项目级文档"
  - pattern: "每月同步|季度同步|里程碑后同步|CONTEXT 失准|架构滞后"
    context: "周期性架构同步，或发现 CONTEXT.md 与代码实际不符"
---

# flow-evolve — 把 change 期间的架构沉淀同步到项目级文档

> **A 前缀表示 Architecture / 横向命令**，不属于任何 change，不写 CHANGE.md / REQUIREMENT.md。直接产出沉淀同步报告 + patch 项目级文档。

## 与 flow-architect 的边界

| 工作流 | 干什么 | 何时跑 | 改 ARCHITECTURE.md 哪段 |
|---|---|---|---|
| **flow-evolve**（本文）| 把 change 级沉淀**单点 append** 到项目级文档 | 每月 / 每季 批量 | 仅 append ADR 列表 / 跨模块契约段 / 修订历史 |
| **flow-architect** | 重写 / 大改 ARCHITECTURE 全篇 | 首次建立 / 重大重构 / ADR 重审 | 任何段都可改 |

如果 flow-evolve 过程中发现需要"改架构"（某条 ADR 应该 deprecated / 依赖规则该改）→ **停下来，提示用户跑 flow-architect**。

## 触发场景

- **每月 / 每季度**：积累若干 change 后批量同步一次
- **里程碑后**：版本发布或大功能完成后，把沉淀凝固到项目层
- **CONTEXT 失准信号**：flow-dev 阶段连续多次发现 AI 没沿用既有抽象（说明索引漏了）
- **`STATE.md` `last_evolve_at` 距今 > 60 天**：flow-go 路由时会主动提示
- **flow-health 冗余巡检留的尾巴**：若 CONTEXT.md「清理窗口专列」或「技术债」段有标记，flow-evolve 扫描时**主动把对应的「既有抽象索引」条目拎出来让用户确认是否删**

## 输入

- `STATE.md`（读 `last_evolve_at` 决定扫描范围）
- `.specs/CONTEXT.md`（patch 目标 · rules 层）
- `.specs/ARCHITECTURE.md`（如存在，则同时是 patch 目标 · structure 层）
- `.specs/archive/<YYYY-MM-DD>-<change-id>/DESIGN.md`（已归档 change 的设计文档 · 只读 § 9 段）
- `.specs/<active-id>/DESIGN.md`（当前活跃 change 也算，但仅在 flow-dev 完成 / flow-integration 之后才纳入）
- `.specs/CHANGELOG.md`（看 change 元信息，做 cross-check）

## 你的职责

### 步骤 1 · 确定扫描范围

读 `STATE.md`：
- 有 `last_evolve_at: <YYYY-MM-DD>` → 只扫该日期**之后**归档的 change
- 无字段（首次跑）→ 扫所有 `.specs/archive/*` 的 DESIGN.md
- 有 `last_evolve_promoted: [<list>]` → 这些 change 的 § 9 已处理过，跳过

输出范围声明（见 `references/EVOLVE.md` 模板）。

如果范围为空 → 直接告诉用户"无新沉淀可同步，CONTEXT.md 已最新"，结束。

### 步骤 2 · 抽取所有 § 9 段

对范围内每个 change：
- grep `^## 9\. 架构沉淀建议`
- 把每个 change 的 § 9 段**原样**收集（带 source change-id 标签）
- 跳过显式写"本 change 无架构层面沉淀建议"的

> Token 预算：每个 change 的 § 9 大约 30-80 行。一般 5-10 个 change 一次同步。超过 15 个建议拆两次跑。

### 步骤 3 · 聚合分类

按五类聚合（跨 change 合并）：

#### 3.1 新增可复用抽象（聚合到一张总表）

| 路径 | 能力 | 来自 change | 复用建议 | 跨 change 冲突？ |
|---|---|---|---|---|
| `src/lib/cache.ts` | LRU 缓存 | add-cache-layer | 沿用 | — |
| `src/utils/date-fmt.ts` | 日期格式化 | add-notifications | 沿用 | ⚠️ 与既有 `src/utils/date.ts` 重复 |

**冲突检测必跑**：每条新抽象 grep CONTEXT.md「既有抽象索引」段是否已有同类 → 有则标 ⚠️。

**GitNexus 路径（可选）**：对每条新抽象调用 `gitnexus_context(name="<抽象名>")`，验证符号在代码中确实存在且仍活跃（不是已删除的过时条目）。如果符号不在 GitNexus 图中 → 标 ⚠️ "代码中可能已不存在"。

#### 3.2 项目级技术决策（聚合）

| 决策 | 取值 | 来自 change | 影响范围 | 与现 CONTEXT 冲突？ |

#### 3.3 跨模块契约（聚合）

```
新增 API：
新增事件：
Schema 变更：
```

#### 3.4 依赖变动（聚合）

| 包 | 版本变动 | 来自 change | 是否替换 |

#### 3.5 禁动清单变动（聚合）

```
新增禁动：
解禁：
```

### 步骤 4 · 逐项 review（用户参与 · **不能跳过**）

按五类**逐项**问用户：

```
🟢 候选 1/N · 新增抽象 · src/lib/cache.ts
   能力：LRU 缓存封装
   来自：add-cache-layer（archive/2026-04-12）
   建议：append 到 CONTEXT.md「既有抽象索引」段
   AI 检测：与 CONTEXT 现有内容无冲突

   选项：
   ✅ 接受 promote
   ❌ 跳过（理由：例如临时用，不长期保留）
   ✏️ 编辑后 promote（你写新版本）

请回复：1 / 2 / 3
```

每条等用户回复后才进下一条。**严禁**一次性 batch promote 全部。

#### 冲突项必须显式问

冲突标 ⚠️ 的项额外问：

```
⚠️ 冲突候选 · 新抽象 src/utils/date-fmt.ts vs CONTEXT 现有 src/utils/date.ts
   两者能力重叠（都是日期格式化）。

   选项：
   ✅ 替换 — 用 date-fmt.ts 替换 date.ts
   ✅ 共存 — CONTEXT 索引同时列两个，注明用途差异
   ❌ 跳过 — 不 promote 新的
   ✏️ 编辑后定夺

请回复：1 / 2 / 3 / 4
```

### 步骤 5 · 生成 patch

收集所有用户批准的项，生成**两份 patch**（项目有 ARCHITECTURE 则两份都生）。

#### 5.1 patch CONTEXT.md（rules 层 · 必生）

见 `references/EVOLVE.md` 模板 §5.1。

#### 5.2 patch ARCHITECTURE.md（structure 层 · 仅在 ARCHITECTURE.md 存在时生）

见 `references/EVOLVE.md` 模板 §5.2。

**两者分工**：
- 技术栈变动 / 抽象索引 / 禁动清单 → 只进 CONTEXT（AI 实施层会读）
- ADR / 跨模块契约 / Schema 总览 / 修订历史 → 只进 ARCHITECTURE（人读 / flow-design 读）
- 项目级技术决策（如"缓存选 Redis"）**两边都进**：CONTEXT 记一句话供 AI 快查，ARCHITECTURE 记完整 ADR

#### 5.3 给用户最终 review

```
✅ 即将应用的 patch（CONTEXT 共 N 条，ARCHITECTURE 共 M 条）：
[先贴 CONTEXT patch]
[再贴 ARCHITECTURE patch · 如有]

确认应用？
1. ✅ 应用两边
2. ⏸️ 暂存 patch 为 .specs/evolve/<YYYY-MM-DD>-EVOLVE-PATCH.md，我手动合
3. ✅ 只应用 CONTEXT
4. ✅ 只应用 ARCHITECTURE
5. ❌ 取消
```

### 步骤 6 · 写入项目级文档（用户选 1/3/4）

**安全要求**：
- 写入前先备份：
  - `cp .specs/CONTEXT.md .specs/CONTEXT.md.bak-<YYYY-MM-DD>`
  - `cp .specs/ARCHITECTURE.md .specs/ARCHITECTURE.md.bak-<YYYY-MM-DD>`（如存在）
- 用 `edit` 工具按段 append / update（**不要整文件 rewrite**）
- 每段 append 在末尾加注释：`<!-- flow-evolve YYYY-MM-DD: from <change-id> -->`
- ADR 编号**顺接现有最大值**（grep `^### ADR-\d+` 取 max + 1）

### 步骤 7 · 输出 EVOLVE 报告 + 更新 STATE

#### 7.1 报告写入 `.specs/evolve/<YYYY-MM-DD>-EVOLVE.md`

见 `references/EVOLVE.md` 模板 §7.1。

#### 7.2 更新 `STATE.md`

```yaml
last_evolve_at: <YYYY-MM-DD>
last_evolve_promoted:
  - add-cache-layer
  - add-bg-jobs
  # ... 本次涉及的所有 change-id（无论是否被 promote，只要扫过就记）
```

## 约束

- **不动业务代码**：只 patch `.specs/` 内项目级文档，禁止改 `src/` / `tests/` / `package.json`
- **逐项 review 强制**：批量 promote 是禁的
- **冲突必显式问**：标 ⚠️ 的项不允许默默走「接受」分支
- **备份必跑**：写任何项目级文档前必须先 `cp` 备份
- **不删已有内容**：只 append / update，不 delete（过时内容建议用户跑 flow-architect，不自动删）
- **不读 § 9 以外的 DESIGN 内容**：避免把 change 级冻结决策错误升级到项目级
- **遇到架构级冲突停下来**：例如某 § 9 要求 deprecate 现有 ADR → 提示用户跑 flow-architect

## 自检

- [ ] 已读 `STATE.md` 的 `last_evolve_at` 决定扫描范围
- [ ] 范围内每个 change 都尝试读了 § 9（即使是"无建议"也已记录跳过）
- [ ] 五类聚合表完整（抽象 / 决策 / 契约 / 依赖 / 禁动清单）
- [ ] 冲突项已检测并显式标 ⚠️
- [ ] 逐项 review 完成，每条都有用户的 1/2/3 回复
- [ ] 检查了 ARCHITECTURE.md 是否存在
- [ ] 最终 patch 已让用户一次性确认
- [ ] 写任何文档前都备份了
- [ ] ADR 编号顺接现有 max + 1
- [ ] EVOLVE 报告已归入 `.specs/evolve/`
- [ ] STATE.md `last_evolve_at` + `last_evolve_promoted` 已更新
- [ ] 遇到架构级冲突未自作主张，已指引用户跑 flow-architect

## 触发下一步

- 同步完成 → 提示用户："下次建议 <YYYY-MM-DD> 同步，或新增 ≥ 5 个 change 后再来"
- 用户选了"暂存 patch 我手动合" → 暂停，告知 patch 路径
- 检测到 ARCHITECTURE.md 不存在但 § 9 里有 ADR 级候选项 ≥ 3 条 → **建议跑 flow-architect 先建立 ARCHITECTURE.md**
- 检测到 ARCHITECTURE.md ADR 冲突 ≥ 5 条 → 建议跑 flow-architect 重审
- 检测到 CONTEXT.md 已积累 ≥ 200 行 → 建议将部分决策迁到 ARCHITECTURE
