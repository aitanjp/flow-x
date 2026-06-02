# 技术栈预选节选

> 完整版见 `reference/tech-stacks.md`（467 行）。本节选仅含 flow-design 步骤 0 所需的「适用矩阵」+「给 AI 展示的标准模板」。

---

## 10 张完整工程模板卡片

### 1. Next.js 全栈（国际 SaaS / 内容站）
- **前端**: Next.js 14+ (App Router) · React 18 · TypeScript 5 · Tailwind CSS v4 · shadcn/ui
- **状态**: Zustand + TanStack Query · Server Components
- **后端**: Next.js Route Handlers / Server Actions
- **ORM**: Drizzle / Prisma
- **数据库**: Postgres（Supabase / Neon）
- **缓存**: Vercel KV (Upstash Redis)
- **鉴权**: Auth.js / Clerk
- **部署**: Vercel
- **适合**: B2B SaaS · 内容/营销站 · 全栈一人开发 · 国际化
- **不适合**: 超高并发后端 · 复杂业务独立部署

### 2. Vite + React + Hono/NestJS（前后端分离 SPA）
- **前端**: Vite 5 · React 18 · Tailwind · shadcn/ui
- **后端**: Hono（Edge）/ NestJS（企业级）/ Express
- **ORM**: Drizzle / Prisma / TypeORM
- **数据库**: Postgres / MongoDB
- **缓存**: Redis
- **鉴权**: Lucia / Better Auth / JWT
- **部署**: 前端 Cloudflare Pages；后端 Railway / Fly
- **适合**: 复杂前端 · 团队前后分工 · 多客户端共后端
- **不适合**: SEO 关键内容站 · 单人项目

### 3. Nuxt 3 全栈（Vue 全家桶）
- **前端**: Nuxt 3 · Vue 3 · TypeScript · UnoCSS
- **状态**: Pinia
- **后端**: Nitro server · Server Routes
- **ORM**: Drizzle / Prisma
- **数据库**: Postgres
- **缓存**: Redis · Nitro Storage
- **鉴权**: nuxt-auth-utils / Better Auth
- **部署**: Vercel · Cloudflare
- **适合**: Vue 团队 · 中文社区 · 轻量 SaaS
- **不适合**: 重 React 生态依赖

### 4. SvelteKit（极简全栈）
- **前端 + 后端**: SvelteKit · Svelte 5 · TypeScript
- **状态**: Svelte Stores（内置）
- **ORM**: Drizzle
- **数据库**: Postgres / SQLite (Turso)
- **缓存**: Redis
- **鉴权**: Lucia
- **部署**: Vercel · Cloudflare
- **适合**: 性能敏感 · 小团队 · 创意站
- **不适合**: 庞大 npm 生态依赖

### 5. FastAPI + React（Python AI / 数据后端）
- **后端**: FastAPI 0.110+ · Pydantic v2 · Python 3.11+
- **ORM**: SQLAlchemy 2 · Alembic
- **数据库**: Postgres + pgvector
- **缓存**: Redis · aiocache
- **AI**: OpenAI SDK · LangChain · LlamaIndex
- **前端**: Vite + React 18 + shadcn/ui
- **部署**: Docker + Railway / Fly
- **适合**: AI/ML 应用 · 数据科学 · 算法/爬虫
- **不适合**: 纯前端 · 实时性极高

### 6. Go + React（高性能后端）
- **后端**: Go 1.22+ · Gin / Echo / Chi
- **ORM**: sqlc / GORM / Ent
- **数据库**: Postgres + pgx
- **缓存**: Redis · BadgerDB
- **鉴权**: golang-jwt · Casbin
- **前端**: Vite / Next + React + shadcn/ui
- **部署**: Docker → K8s
- **适合**: 高 QPS API · 金融 · 基础设施 · 实时系统
- **不适合**: MVP 快速迭代 · 业务频繁变化

### 7. Spring Boot + Vue 中后台（国内企业经典）
- **后端**: Spring Boot 3 · Java 17+ · MyBatis Plus
- **数据库**: MySQL 8 / Postgres + Druid
- **缓存**: Redis + Redisson
- **鉴权**: JWT + Spring Security / Sa-Token
- **队列**: RocketMQ / RabbitMQ / Kafka
- **前端**: Vue 3 + Vite + TypeScript · Element Plus
- **部署**: Docker + Nginx · K8s
- **适合**: 国内 B2B/政务/金融 · 中后台 dashboard
- **不适合**: 初创快速迭代 · 国际化 SaaS

### 8. NestJS + Next.js（TypeScript 全栈分离）
- **后端**: NestJS 10 · Prisma · Postgres
- **缓存**: Redis
- **鉴权**: Passport + JWT
- **前端**: Next.js 14 · React 18 · shadcn/ui
- **部署**: 前端 Vercel；后端 Railway / Fly / Docker
- **适合**: 企业级 TypeScript 全栈 · 共享 DTO · 复杂领域
- **不适合**: 轻量项目 · 团队无 TS 经验

### 9. Laravel + Vue / React（PHP 全栈）
- **后端**: Laravel 11 · PHP 8.3
- **ORM**: Eloquent
- **数据库**: MySQL / Postgres
- **缓存**: Redis
- **鉴权**: Laravel Sanctum / Fortify
- **前端**: Vue 3 + Inertia.js / React + Inertia
- **部署**: Forge / Vapor / Docker
- **适合**: PHP 团队 · 快速开发 · 成熟生态
- **不适合**: 高并发 · 实时系统

### 10. Tauri + React/Vue/Svelte（桌面端）
- **前端**: React / Vue / Svelte
- **后端**: Rust（Tauri runtime）
- **数据库**: SQLite（嵌入）/ 远程 API
- **部署**: 桌面安装包（.exe/.dmg/.AppImage）
- **适合**: 桌面工具 · 本地优先 · 跨平台
- **不适合**: 纯 Web · 需要服务端渲染

---

## 给 AI 在 2-design 阶段展示用的标准模板

```
技术栈预选

我为你列出 5~6 张最匹配的完整工程模板，请选一个数字回复：

1. <名称> — <一句话定位>（<适合场景>）
2. <名称> — <一句话定位>（<适合场景>）
3. ...

推荐：
   - 首选：<编号><名称> — 因为 <结合 AC + 非功能需求的一句话理由>
   - 备选：<编号><名称> — 因为 <理由>

回复数字即可（如"1"），或描述偏好。
```

**规则**：
- 只列 5~6 张，不列全部
- 必给 1 首选 + 1 备选
- 推荐理由必须结合 REQUIREMENT.md 的 AC 和非功能需求
- 显式排除 1~2 个 + 理由
