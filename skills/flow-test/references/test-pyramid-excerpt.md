# 测试金字塔节选

> 完整版见 `reference/test-pyramid.md`（~200 行）。本节选仅含 flow-test 所需的「适用矩阵」+ 各轮核心标准。

---

## 适用矩阵（按项目类型裁剪）

| 轮次 | Web 前端 | 后端 API | CLI/库 | 全栈 | 移动端 |
|---|---|---|---|---|---|
| 1 功能 | ✅ 必跑 | ✅ 必跑 | ✅ 必跑 | ✅ 必跑 | ✅ 必跑 |
| 2 性能 | ✅ 必跑 | ✅ 必跑 | ⚠️ 部分 | ✅ 必跑 | ⚠️ 部分 |
| 3 安全 | ✅ 必跑 | ✅ 必跑 | ⚠️ 部分 | ✅ 必跑 | ✅ 必跑 |
| 4 兼容 | ✅ 必跑 | ⚠️ 部分 | ❌ 跳过 | ✅ 必跑 | ✅ 必跑 |
| 5 可观测 | ⚠️ 部分 | ✅ 必跑 | ❌ 跳过 | ✅ 必跑 | ⚠️ 部分 |

---

## 第 1 轮 · 功能测试

### 工具选择（按栈）

| 栈 | 单测 | 集成 | E2E |
|---|---|---|---|
| Web 前端 | Vitest/Jest + Testing Library | MSW + Vitest | Playwright / Cypress |
| Node 后端 | Vitest/Jest | supertest + 容器 DB | Playwright API / Postman |
| Python | pytest | pytest + testcontainers | pytest + httpx |
| Go | testing | testify + dockertest | httptest |
| Rust | cargo test | testcontainers-rs | reqwest + tokio |

### 通过标准
- 每条 AC >= 1 条测试或 UAT
- 关键路径行覆盖 >= 80%（core >= 90%）
- 错误路径有显式测试
- 边界值 >= 3 条

### 反模式
- 只测实现细节（如 mock 后断言 mock 被调用过）
- 测试名是 `test_function_works`
- 一个测试断言 > 5 条
- 共享状态的测试
- Sleep / setTimeout 等待

---

## 第 2 轮 · 性能测试

### 前端
| 工具 | 测什么 | 通过标准 |
|---|---|---|
| Lighthouse CI | LCP / CLS / INP / TBT | LCP < 2.5s / CLS < 0.1 / INP < 200ms |
| Bundle Analyzer | 主包 / 路由分包 | 主包 < 200KB gzip |

### 后端
| 工具 | 测什么 | 通过标准 |
|---|---|---|
| k6 / locust / wrk | 关键 API 在 N 倍 QPS 下 | p95 < SLO；错误率 < 0.1% |
| EXPLAIN ANALYZE | 慢查询审计 | 关键查询 < 100ms / 无 N+1 |

### 反模式
- 只测 happy load，不测 spike / sustained
- 在开发机跑性能测试
- 单次跑就下结论
- 没有基线对比

---

## 第 3 轮 · 安全测试

### 依赖漏洞
| 栈 | 工具 | 通过标准 |
|---|---|---|
| Node | `npm audit --production` / Snyk | 无 high / critical |
| Python | `pip-audit` / `safety check` | 无 high / critical |
| Go | `govulncheck ./...` | 无已知漏洞 |

### 秘钥扫描
`trufflehog filesystem .` — 通过：0 命中。命中必须立即 rotate。

### SAST
Semgrep / CodeQL / Bandit — 无 high；medium 有处理记录。

### OWASP Top 10
逐项标 ✅ / ❌ / 🟡。

---

## 第 4 轮 · 兼容性测试

### 前端跨浏览器
- 桌面：Chrome / Firefox / Safari / Edge（最新 2 版本）
- 移动：iOS Safari / Android Chrome
- 视口：360 / 768 / 1024 / 1440

### 数据迁移（涉及 schema 变更必跑）
- 生产数据快照预演
- 实测耗时 → 决定 maintenance window
- 回滚脚本（down）就位且测过
- 加 NOT NULL：旧行 backfill 已验证
- 改字段类型：cast 不丢数据

---

## 第 5 轮 · 可观测性验证

### 日志
- 关键路径入口/出口/异常都有 log
- 含 trace-id，结构化 JSON
- 不含 PII / 秘钥 / token

### 指标/追踪
- 业务关键 metric 有打点
- RED 指标覆盖关键 endpoint
- 跨服务 trace 串通

### 告警 + 健康检查
- 关键失败有告警 + runbook 链接
- `/health` 区分 liveness / readiness
- 无噪音告警
