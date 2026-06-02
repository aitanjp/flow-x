---
name: flow-test
description: 五轮测试金字塔执行器。基于 REQUIREMENT.md 的 AC 和 DESIGN.md 的技术栈，执行功能/性能/安全/兼容/可观测五轮测试，每轮按项目类型可裁剪，输出 TEST.md 报告。Use when 用户确认所有 DEV 任务完成后需要进入测试阶段，或需要补测试/覆盖率审计时。
---

# flow-test — 五轮测试金字塔

## Goal

测试不是"跑一下单测"，是 5 个维度的金字塔。每轮按项目类型可裁剪。

## Workflow

### 步骤 0 · 声明本次走哪几轮（强制）

在 TEST.md 开头显式输出适用矩阵：

```
| 轮次 | 状态 | 范围 | 跳过理由 |
|---|---|---|---|
| 第 1 轮 · 功能 | ✅ 必跑 | 全部 AC | — |
| 第 2 轮 · 性能 | ✅ / ⚠️ / ❌ | ... | ... |
| 第 3 轮 · 安全 | ✅ / ⚠️ / ❌ | ... | ... |
| 第 4 轮 · 兼容 | ✅ / ⚠️ / ❌ | ... | ... |
| 第 5 轮 · 可观测 | ✅ / ⚠️ / ❌ | ... | ... |
```

禁止没声明就跳过任何轮次。每个 ❌ 必须有理由。

### 第 1 轮 · 功能测试

#### 1.1 测试矩阵
每条 AC 映射到测试用例。强制：每条 AC >= 1 条覆盖。空缺必须解释。

#### 1.2 UAT 脚本
无法自动化的 AC 写 UAT 脚本（前置/步骤/期望/通过失败）。

#### 1.3 覆盖率与边界
- 跑 `<test-cmd> --coverage`，贴输出
- 关键路径行覆盖 >= 项目门槛（默认 80%；core 90%）
- 边界值用例（空/极大/极小/Unicode/null）>= 3 条
- 错误路径必须有显式测试

#### 1.4 测试质量自检 · 6 维测试衰退风险

| 编号 | 衰退风险 | 诊断问题 |
|---|---|---|
| T1 | Test Obscurity | 读这个测试能马上看出「它在验证什么」吗？ |
| T2 | Test Brittleness | 重构实现会让这个测试坏掉吗？ |
| T3 | Test Duplication | 同一个场景是否被多个测试换个姿势验证？ |
| T4 | Mock Abuse | mock 是否遮蔽了真实问题？ |
| T5 | Coverage Illusion | 覆盖率高但 assertion 空？ |
| T6 | Architecture Mismatch | 测试层级是否与架构匹配？ |

路径 A（装了 brooks-lint）：`/brooks-test`，输出 4 要素格式贴入 TEST.md。
路径 B（未装）：逐个维度检查，命中 >= 3 项 → release 前必修。

### 第 2 轮 · 性能测试

#### 2.1 性能预算确认
从 REQUIREMENT.md「非功能性需求」提取性能预算。**没有就停下来**，让用户先补。

#### 2.2 前端性能（Web 项目）
- Lighthouse CI：LCP / CLS / INP / TBT
- Bundle Analyzer：主包 + 路由分包大小
- 与上一版基线对比，**禁止退步**

#### 2.3 后端/API 性能
- k6/locust 在 N 倍业务 QPS 下：p95 / p99 / 错误率
- 数据库慢查询审计（`EXPLAIN ANALYZE` 关键查询）
- 检测 N+1（ORM 项目必查）

#### 2.4 通过标准
逐项对照预算，输出"✅ 达标 / ❌ 退步 X% / ⚠️ 接近阈值"，**不允许"性能良好"空话**。

### 第 3 轮 · 安全测试

#### 3.1 依赖漏洞扫描
`npm audit --production` / `pip-audit` / `govulncheck`。通过：无 high/critical。

#### 3.2 秘钥扫描
`trufflehog filesystem .`。通过：0 命中。命中必须立即 rotate。

#### 3.3 静态扫描（SAST）
Semgrep / CodeQL / Bandit。无 high；medium 有处理记录。

#### 3.4 OWASP Top 10 清单
逐项标 ✅ 已测 / ❌ 不适用 / 🟡 待补。

### 第 4 轮 · 兼容性测试

#### 4.1 前端跨浏览器/跨设备
- 桌面：Chrome / Firefox / Safari / Edge（最新 2 版本）
- 移动：iOS Safari / Android Chrome
- 视口：360 / 768 / 1024 / 1440

#### 4.2 数据迁移测试（涉及 schema 变更必跑）
- 迁移文件路径已 trace（来自 SUMMARY「数据库迁移」段）
- 在生产数据快照上预演迁移脚本
- 实测耗时 → 决定是否需要 maintenance window
- 回滚脚本（down）就位且测过：up → down → 复原 → 再 up
- 双写期/灰度方案有验证步骤
- 加 NOT NULL 字段：旧行 backfill 已验证
- 改字段类型：cast 不丢数据/不截断/不溢出已验证

#### 4.3 跨版本/跨编码
- 旧 schema 数据能否被新代码正确读写
- API 版本兼容
- UTF-8 / 不同 locale

### 第 5 轮 · 可观测性验证

#### 5.1 日志验证
- 关键路径入口/出口/异常都有 log
- 含 trace-id，结构化（JSON）
- **不含 PII/秘钥/token**（grep 验证）
- 错误日志含足够上下文

#### 5.2 指标/追踪
- 业务关键 metric 有打点
- RED 指标覆盖关键 endpoint
- 跨服务 trace 串通（如有分布式调用）

#### 5.3 告警 + 健康检查
- 关键失败有告警 + runbook 链接
- `/health` 区分 liveness / readiness
- 无噪音告警

### 步骤 N · 回归测试登记

本次新加/修复的测试用例统一登记到 TEST.md 末尾。

## Constraints

- 测试用例从 AC 派生，不从实现派生
- Bug 修复必伴随回归测试，加入 LESSONS
- 禁止通过删除/弱化测试来"修复"失败
- 不允许跳过任何轮次而没有显式理由
- 性能/安全/兼容轮次的"通过/失败"必须基于可量化指标或工具输出
- AC 覆盖优先于行覆盖

## Validation

- [ ] 范围声明已输出，5 轮状态都明确
- [ ] 第 1 轮：每条 AC 有覆盖 + 覆盖率达标 + 边界 >= 3 + 测试质量 6 维自检过
- [ ] 第 2 轮：性能预算逐项对比 + 与上版基线对比
- [ ] 第 3 轮：依赖/秘钥/SAST/OWASP 各有处理记录
- [ ] 第 4 轮：跨浏览器/数据迁移/跨版本按需跑过
- [ ] 第 5 轮：日志/指标/告警/健康检查清单逐项验证
- [ ] 跳过的轮次都有理由
- [ ] 测试新增的用例已登记

## Resources

- `references/TEST.md` — 产出模板（含 5 轮报告段）
- `references/test-pyramid-excerpt.md` — 5 轮工具/标准/反模式矩阵节选
