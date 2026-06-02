---
name: <项目名>
description: <一句话美学定位>

# 所有颜色用 OKLCH。Hex 仅在已有遗留约束时允许，必须解释。
colors:
  brand: "oklch(% chroma hue)"
  brand-deep: "oklch(...)"
  bg: "oklch(...)"
  surface: "oklch(...)"
  text-primary: "oklch(...)"
  text-secondary: "oklch(...)"
  text-tertiary: "oklch(...)"
  border: "oklch(...)"

typography:
  display:
    fontFamily: "<具体字体>, <fallback>, serif/sans-serif"
    fontSize: "clamp(2.5rem, 7vw, 4.5rem)"
    fontWeight: <100~900>
    lineHeight: 1
    italic: <true|false>
  headline: { ... }
  title: { ... }
  body:
    fontFamily: "..."
    fontSize: "1rem"
    fontWeight: 400
    lineHeight: 1.6
  body-lead: { ... }
  supporting: { ... }
  label:
    fontWeight: 500
    textTransform: uppercase
    letterSpacing: "0.05em"
  micro-label: { ... }
  mono: { ... }

spacing:
  xs: "8px"
  sm: "16px"
  md: "24px"
  lg: "32px"
  xl: "48px"
  "2xl": "80px"
  "3xl": "120px"

rounded:
  none: "0"
  sm: "4px"
  md: "8px"
  lg: "12px"
  xl: "16px"

motion:
  ease-out: "cubic-bezier(0.16, 1, 0.3, 1)"
  ease-out-quint: "cubic-bezier(0.22, 1, 0.36, 1)"
  duration-fast: "150ms"
  duration-base: "300ms"
  duration-slow: "600ms"

shadow:
  hover-lift: "0 4px 24px -4px rgba(0,0,0,0.12), 0 1px 3px rgba(0,0,0,0.06)"
  card-lifted: "0 20px 40px rgba(0,0,0,0.08)"
  accent-glow: "0 20px 60px var(--color-brand-veil)"
---

# UI Design: <项目名>

## 0. 视觉语汇对齐（仅 brownfield 填 · greenfield 删掉此段）

### 0.1 观察报告

- **Token 源**：<文件路径>
- **主色实际比例**：<占 X%；在哪些元素上用>
- **中性色**：<背景 / 文字 / 边框 oklch(...)>
- **hover / focus 反馈**：<具体变化>
- **动效语言**：<cubic-bezier / duration 档位>
- **elevation 层级**：<几级阴影>
- **卡片密度 / rounded**：<内边距 / 圆角档>
- **图标库**：<lucide / heroicons / 自绘>
- **文案调性**：<工程向 / 营销向 / 中性>

### 0.2 用户校准结论

- <用户确认 / 修正>

### 0.3 应用策略

- 沿用：<哪些维度照搬>
- 延伸：<哪些维度在原基础上扩>
- 打破：<哪些维度刻意远离>

## 1. 美学北极星

> 一段话写清视觉北极星。

### v0 确认摸路

- **已确认的假设**：
  - <例：hero 有背景图，用 16:9 占位符>
- **用户指出的偏差**：
  - <例：hero 不应该整屏 → 已改>

## 2. 4 个决策问题

- **目的**：<解决什么问题？给谁用？核心动作？>
- **调性**：<具体名称>
- **约束**：<技术栈 / 性能 / 无障碍 / 品牌>
- **差异化**：<让人记住的那一件事>

## 3. 颜色系统

### Primary

- **<主色名>** `oklch(...)`：用于 <场景>。**绝不用于** <避免场景>

### Neutral

- **<bg>** `oklch(...)`：主背景
- **<text>** `oklch(...)`：主文字

### 命名规则

- **The One Voice Rule**：主色只有 1 个
- **The Tinted Neutral Rule**：所有中性色含 chroma，不用纯灰

## 4. 字体系统

- **Display**：<字体名>。**为什么不用 Inter/Roboto**：<理由>
- **Body**：<字体名>
- **Mono / Label**：<字体名>

### 层次表

| 角色 | 字体 | 字号 | 字重 | 行高 | 备注 |
|---|---|---|---|---|---|
| Display | ... | clamp(...) | ... | ... | hero |
| Headline | ... | ... | ... | ... | section |
| Body | ... | 1rem | 400 | 1.6 | 段落 |

## 5. 间距 & 圆角 & 动效

按 frontmatter token 落实。禁止在 token 之外引入新数值。

## 6. 关键组件规约

### Button (Primary)

- **形状**：<sharp/rounded>
- **背景**：<bg token>
- **文字**：<text token>
- **内边距**：<spacing tokens>
- **at rest**：<阴影/边框>
- **hover**：<具体变化>
- **focus**：<focus ring>

### Input / Field

- **at rest**：<边框/背景>
- **focus**：<边框色变化>
- **error**：<错误状态表达>

### Card / Container

- **at rest**：平面 / 阴影 / hairline
- **hover**：响应
- **嵌套规则**：是否允许 card 套 card（默认禁止）

### Navigation

- 高度、字体、间距、hover/active

### Typography Hierarchy

参考第 4 节层次表。

## 7. Do's and Don'ts（本项目特定）

### Do

- ...

### Don't

- ...

## 8. 占位符策略

| 缺的东西 | 本项目有什么？ | 缺时用什么占位 | 禁什么 |
|---|---|---|---|
| 图标 | | [icon] 方块 | emoji / AI 粗糙 SVG |
| 头像 | | 首字母圆 + 品牌色 | AI 生人脸 |
| 图片 | | aspect-ratio 卡片 | stock photo |
| 数据 | | **反问用户** | 编数字 |
| logo | | 品牌名文字标签 | AI 自绘图 |
| 客户推荐 | | TESTIMONIAL PLACEHOLDER | 编造 |

## 9. 反 AI-slop 自检结果

- [ ] 字体类禁忌：未命中
- [ ] 颜色类禁忌：未命中
- [ ] 阴影类禁忌：未命中
- [ ] 边框类禁忌：未命中
- [ ] 动效类禁忌：未命中
- [ ] 布局类禁忌：未命中
- [ ] 文案类禁忌：未命中
- [ ] 组件类禁忌：未命中

## 10. 触发任务

下一步进入 flow-task 时，把以下作为第一批 UI 任务：

- T-UI-01：把 design tokens 物化为 CSS variables
- T-UI-02：实现 typography 层次
- T-UI-03：实现 Button (Primary)
- T-UI-04：实现 Input
- T-UI-05：实现 Card
