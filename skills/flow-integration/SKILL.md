---
name: flow-integration
description: 集成验证与归档执行器。跑全套自动化测试，引导人工 UAT，失败诊断并产 fix-plan（最多 3 轮自动重试），提名 LESSONS 到知识库，归档 change 到 archive/ 目录并更新 CHANGELOG.md 和 STATE.md。Use when 用户确认 REVIEW.md 通过后需要进入集成验收阶段，或需要 UAT/归档/发布时。
---

# flow-integration — 集成验证 + UAT + 失败诊断 + 归档

## Goal

作为 Verifier + Release，完成集成验证、人工 UAT、失败诊断与修复、LESSONS 提名、归档。

## Workflow

### 1. 跑全套自动化

- 全量单测：`npm test`（或等价）
- 集成测试/e2e：`npm run e2e`（如有）
- 类型检查/静态检查：`tsc --noEmit` / `lint` 等
- 构建：`npm run build`

**贴出每条命令的真实输出**。任何失败立即进入「失败诊断」。

### 2. 引导人工 UAT

逐条读 TEST.md 的 UAT 脚本，向用户提问：

```
UAT-1：<场景>
  前置：...
  步骤：1. ... 2. ... 3. ...
  通过 / 失败 / 描述问题：
```

记录每条 UAT 结果到 `.specs/<id>/UAT.md`。

### 3. 失败诊断（自动 + 人工）

任何失败（自动测试或 UAT）：
1. 切到「Diagnose 子角色」，定位 root cause（不是症状）
2. 产出 fix-plan：追加到 TASK.md，编号 `T-FIX-XX`，含完整 verify
3. 回到 flow-dev 执行修复
4. 修完回到本步重跑

**R2.6**：自动重试 <= 3 轮。第 3 轮仍失败必须停下来要求人工决策。

### 4. 提名 LESSONS（ARCHIVE 之前必跑 · R1.8）

扫本次 change 的所有 `*-SUMMARY.md`「决策与偏离」段，以及任何遗留的 `*-PROGRESS.md`「已排除方案」段。

按提名条件筛选：
- 调试/试错耗时 > 30 分钟 → 提名
- 错因不局限于本任务、其它任务也会撞 → 提名
- 6 个月内有合理概率被再次尝试 → 提名
- 否则不入库（避免污染）

把入选的失败按 LESSONS.md 条目格式追加到 `.specs/LESSONS.md`，编号续上 `L-NNN`。
复核现有 active 条目，看是否有本次 change 让它们 `superseded` 或 `deprecated`。

### 5. 归档（ARCHIVE）

全部通过后：
- 把 `.specs/<change-id>/` 移动到 `.specs/archive/<YYYY-MM-DD>-<change-id>/`
- 在 `.specs/CHANGELOG.md` 追加一行（日期/change-id/摘要/PR 链接/新增 LESSONS 条目编号）
- 更新仓库根 `STATE.md`
- **不要归档 `.specs/LESSONS.md`**——项目级常驻文件

#### 5.1 项目级架构文档同步（不在本步做 · 走 A-evolve）

本 change 的 `DESIGN.md §9` **不在归档时立即合并到 CONTEXT.md**。原因：单个 change 视角窄，容易把临时决策错升项目级。

正确做法：
- 归档时只把 DESIGN.md 原样移入 archive/（§9 内容随之冻结）
- 提示用户：积累 >= 5 个 change 或满 60 天后跑 flow-evolve 批量同步
- **禁止**在本步直接修改 `.specs/CONTEXT.md`

### 6. 出 PR（可选）

如果用户用 git 流水线：
- PR 标题/正文从 CHANGE.md + SUMMARY.md 自动拼装
- 列出涉及的文件、AC 覆盖、UAT 结论
- `.specs/` 内文件归类到 PR 描述

## Constraints

- UAT 失败的自动重试 <= 3 轮
- 禁止声称"通过"而没贴真实输出
- 归档操作必须用户确认后才执行

## Validation

- [ ] 全量自动化结果已贴出且全绿
- [ ] 每条 UAT 都有人工通过/失败标注
- [ ] 失败的项目都已经过最多 3 轮自动重试，超限的已暂停
- [ ] CHANGELOG 已追加
- [ ] 归档目录已创建（用户确认后）

## Resources

- `references/LESSONS.md` — 提名条件与条目格式
