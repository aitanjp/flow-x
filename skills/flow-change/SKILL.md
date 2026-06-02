---
name: flow-change
description: 变更提案生成器。把用户的模糊想法通过反问澄清为一份变更提案 CHANGE.md，自动生成 change-id，检测架构级变更，前端项目预选视觉调性，判定影响面并给出路径建议。Use when 用户说"我想做/加/实现/设计某事"且无活跃 change 时，或需要发起新变更提案时。
---

# flow-change — 变更提案

## Goal

把用户的模糊想法通过结构化反问澄清为一份变更提案，并判定影响面、给出后续路径建议。

## Workflow

### 0. 自动生成 change-id

- 从用户描述提取核心关键词，转成 kebab-case（小写、短横线分隔、英文）
- 长度 2~4 个词。例：「设计陪诊网站」→ `companion-platform`；「加个深色模式」→ `dark-mode`
- 检查 `.specs/<id>/` 不存在，若冲突自动加序号 `<id>-2`、`<id>-3`
- 在反问的第一条消息里向用户显式声明：
  ```
  自动生成 change-id：`<id>`（不满意请告诉我新的，否则继续。）
  ```
- 用户没纠正就照用，**不要等确认**

### 0.4. 架构级变更检测（必跑）

**目的**：检测是否涉及项目级架构变更（拆服务 / 换数据库 / 换鉴权方案等），这类变更应先在 ARCHITECTURE 层达成共识。

**命中任一即触发**：
1. 影响项目级模块结构（新增/拆分/合并/删除模块）
2. 影响已有 ADR（与 ARCHITECTURE.md §3 冲突或要新增不可逆决策）
3. 改公共契约（API 风格 / 事件总线协议 / Schema 主键策略）
4. 影响容量边界（触发 ARCHITECTURE.md §6 预警阈值）
5. 跨服务编排（容器化 / 上云 / 多区域 / 多租户）

**明确不算架构级**：在既有架构内加新 feature / 业务模块；在既有抽象上加适配器；bug 修复 / UI 改版 / 文案改动。

**命中后反问用户**：
```
检测到本次涉及项目级架构变更。选项：
1. 先跑 flow-architect（推荐 · 重审 ADR + 更新模块图）
2. 我清楚影响，继续 flow-change（DESIGN 会显式声明对 ADR-NNN 的影响）
3. 这其实是 change 级局部改造（请说明理由，我重判）
```

- 选 1：暂停本 skill，引导用户跑 flow-architect。已生成的 change-id 暂留，完成后复用
- 选 2：继续，但强制在 CHANGE.md 末尾加「架构层影响声明」
- 选 3：用户给解释，AI 重新判断

### 0.5. 前端项目识别

描述包含以下任一 → 判定为前端项目：网站/网页/页面/web/app/应用/移动端/小程序/dashboard/后台/界面/UI/前端/用户端/客户端/GUI

非前端项目（CLI / 后端 API / lib / SDK）跳过 0.6。

### 0.6. 视觉调性预选（仅前端项目，独立一条消息，等用户回复）

**红线：本步不出其他问题。只列调性卡片 + 推荐，用户选定后才开启反问。**

加载 `references/ui-aesthetics-excerpt.md` 的「调性」一节，按标准模板呈现 9 张卡片：
- 项目名 + 一句话业务描述
- 9 张卡片（编号 + 名称 + 关键词 + 3 个参考产品）
- **必给 1 首选 + 1 备选**，理由结合业务
- **显式排除**明显不合适的 1~3 个 + 理由
- 末尾：`请回复数字（如 "6"）或描述你想要的感觉`

**例外**：用户描述含强偏好（"参考 Notion"）→ 直接锁定对应调性，跳过卡片。

**选定后**：写入 CHANGE.md「视觉调性」字段，继承到 2a-ui-design。

### 1. 反问

用户选定调性后（或非前端跳过 0.6 后），用结构化提问把"为什么/给谁/解决什么/何时算完"问清楚。每轮最多 3 个问题，等用户回答再继续。
- **不允许与调性卡片同屏呈现**

### 2. 影响面判定

- 是否需要新增/修改 REQUIREMENT.md？
- 是否触及架构（需要更新 DESIGN.md / 新增 ADR）？
- 是否影响现有 AC？

### 3. 范围排除

明确写出**这次不做什么**。至少 1 条。

### 4. 生成 CHANGE.md

使用 `references/CHANGE.md` 模板，填好后保存到 `.specs/<change-id>/CHANGE.md`。

### 5. 路径建议

- **完整**：REQUIREMENT → DESIGN → TASK → DEV → TEST → REVIEW → INTEGRATION
- **中等**：(REQUIREMENT 增量) → TASK → DEV → TEST → REVIEW → INTEGRATION
- **最短**：TASK → DEV → TEST → REVIEW → INTEGRATION（仅纯 bug 修复或微调）

## Constraints

- 不允许跳过反问直接出方案
- 不允许凭空假设未确认的需求点
- 不允许在 CHANGE.md 里写实现细节（那是 DESIGN 的事）
- 步骤 0.4 已跑：未命中就跳过；命中且用户选 2 → CHANGE.md 末尾有「架构层影响声明」

## Validation

- [ ] CHANGE.md 包含：Why / What / 影响面 / 范围排除 / 验收线
- [ ] 至少明确写出 1 条「本次不做」
- [ ] 路径建议有理由，不是默认全跑
- [ ] 没有跳到实现层
- [ ] 步骤 0.4 已跑且处理正确

## Resources

- `references/CHANGE.md` — 产出模板
- `references/ui-aesthetics-excerpt.md` — 调性卡片源（仅「给 AI 在 0-change 阶段展示用的标准模板」节）
