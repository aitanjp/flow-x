# 架构演进同步 · YYYY-MM-DD

## 扫描范围
- 起始：上次同步 <date> 之后
- 涵盖 change：[列表]

## 候选汇总
- 新抽象：M 条 · 接受 X · 跳过 Y · 编辑 Z
- 技术决策：M 条 · ...
- 跨模块契约：...
- 依赖变动：...
- 禁动清单：...

## 应用 patch

### CONTEXT.md patch · YYYY-MM-DD

#### 「既有抽象索引」段 append
+ src/lib/cache.ts · LRU 缓存 · 来源 add-cache-layer (2026-04-12)
+ src/lib/queue.ts · 任务队列 · 来源 add-bg-jobs (2026-04-20)

#### 「已锁技术决策」段 append
+ 缓存层：Redis（替代原 in-memory）· 来源 add-cache-layer
+ 后台作业框架：BullMQ · 来源 add-bg-jobs

#### 「技术栈」段 update
~ ioredis 5.4.0 替换 node-redis（旧版可拆）
+ bullmq 5.x 新增

#### 「禁动清单」段 update
+ src/lib/cache.ts 不允许绕过直接 import ioredis
- src/legacy/cache-old.ts（标 deprecated · 待拆）

### ARCHITECTURE.md patch · YYYY-MM-DD（仅当 ARCHITECTURE.md 存在时）

#### § 3 ADR 列表 · 新增
+ ADR-008 · 缓存层：Redis
  - 状态：accepted (YYYY-MM-DD)
  - 取舍：in-memory LRU / Redis / Memcached
  - 决定：Redis 7（ioredis 客户端）
  - 理由：多实例部署需共享缓存
  - 代价：多一个运维组件
  - 来源：add-cache-layer
  - 推翻成本：中

#### § 4.1 公共 HTTP API · append
+ /api/cache/*  ← add-cache-layer
+ /api/notifications/*  ← add-notifications

#### § 4.2 事件总线 · append
+ events.cache.invalidated  · 来源 add-cache-layer

#### § 4.3 数据库 schema · 提及新表
+ cache_entries（详细 schema 见 add-cache-layer migration）

#### § 8 修订历史 · append
| YYYY-MM-DD | flow-evolve | ADR-008 新增（来自 add-cache-layer）| flow-evolve |

## 跳过项 + 理由
- src/utils/date-fmt.ts → 跳过，理由：与既有 date.ts 重复，保留单一源

## 下次建议同步时间
建议 <YYYY-MM-DD>（约 60 天后），或在新增 ≥ 5 个有 § 9 内容的 change 之后
