---
name: flow-ui-design
description: UI 美学导演。为前端项目产出 UI-DESIGN.md，含美学北极星、OKLCH design tokens、5 维决策（字体/颜色/动效/空间/质感）、关键组件规约、占位符策略和反 AI-slop 自检。Use when 前端/可视化项目确认 DESIGN.md 后需要进入 UI 设计阶段，或需要定义/更新 design system 时。后端/CLI/lib 项目跳过此阶段。
---

# flow-ui-design — UI 美学设计

## Goal

为前端项目决策"看起来怎样"，产出可直接落地的 design tokens 和组件规约。

## Workflow

### 0. 判定 greenfield vs brownfield

- **greenfield**：空项目 / 第一个 UI change / 用户明说"重新做 UI" → 跳过 1.5，直接进 1
- **brownfield**：已存在 UI，本次是加新页面/组件/改现有页面 → **必走 1.5 视觉语汇对齐**

### 1. 美学方向决策（greenfield 最重要）

按 4 个问题显式回答，缺一不可：
- **目的**：界面解决什么问题？谁用？核心动作？
- **调性**：从 CHANGE.md「视觉调性」段读取已选项。**不允许在本阶段让用户重选。**
  - 无该字段（旧项目/跳过 0-change）→ 才按调性卡片让用户选
  - 用户主动说"想换调性" → 引导回 0-change 改 CHANGE.md
- **约束**：技术栈 / 性能预算 / 无障碍等级 / 品牌限定
- **差异化**：让人记住的"那一件事"是什么？

**反问**：上述 4 题任一答不出 → 停下来反问。

### 1.5 brownfield 视觉语汇对齐（旧项目加 UI 必走）

#### 1.5.1 挖出现有视觉语汇

优先读代码 >> 看截图。按顺序提取：
1. Token 源：grep tokens / theme / tailwind.config / :root
2. 实际色彩比例：打开 3-5 个代表页面记录
3. 交互反馈语言：grep hover / focus / transition
4. 动效语言：grep cubic-bezier / ease- / duration-
5. 结构语言：elevation 层级、卡片密度、rounded、布局模式
6. 图形与图标：图标库、插画风格
7. 文案调性

形成「观察报告」贴给用户校准。

#### 1.5.2 用户确认后再进步骤 2

- 用户修正 → 记录修正，重贴确认
- 用户说没问题 → 5 维决策默认势能调成"沿用观察到的"
- 用户明确说"本次想远离原有风格" → 保留观察作为约束边界

### 2. 美学维度决策

按 5 维度逐项决策，每个必须给理由：
- **字体**：display + body，写出具体名称，**显式避开** Inter/Roboto/Arial/Helvetica/system-ui
- **颜色**：用 OKLCH 写出主色 + 中性梯度
- **动效**：缓动曲线 + 时长档位
- **空间**：间距 scale（6~8 个非线性数值）+ 圆角词汇（<= 5 个值）
- **质感**：背景策略

### 3. v0 草稿确认（开工前纠偏）

v0 = 占位符 + 关键布局 + 已选 token + 假设清单。30 秒内能读完的极简 Markdown。

贴给用户三种回复：
- "可以 go" → 进步骤 4
- "某维度偏了" → 仅改该维，重贴 v0
- "全推了重来" → 回步骤 1-2

### 4. Design Tokens

写出可直接复制为 CSS variables / Tailwind config / theme file 的 token 定义。所有颜色 OKLCH，间距用 px/rem。

### 5. 关键组件规约

至少定义：Button / Input / Card / Navigation / Typography hierarchy

### 6. Do's and Don'ts

引用 `references/ui-anti-patterns.md`，列出本项目的额外禁忌。

### 7. 占位符策略

| 缺的东西 | 正确做法 | 禁止做法 |
|---|---|---|
| 图标 | [icon] 方块 / 简几何形 | emoji / AI 粗糙 SVG |
| 头像 | 首字母圆形 + 品牌色 | AI 生人脸 |
| 图片 | aspect-ratio 占位卡片 | stock photo / AI 生成 |
| 数据 | **反问用户要真数据** | 编数字 |
| logo | 文字标签 + 简几何形 | AI 自绘图形 |

**红线**：emoji 只有在指定品牌本就用 emoji 时才允许，默认一个不用。

### 8. 反 AI-slop 自检

输出 UI-DESIGN.md 前，逐条对照 `references/ui-anti-patterns.md` 的"强制禁忌"段，确认未命中。命中必须改或显式标注例外理由。

## Constraints

- **R3.1 延伸**：禁止给完整组件实现代码（CSS 片段示例 OK，整个组件 < 30 行 OK）
- 颜色必须 OKLCH（除非遗留约束并声明）
- 字体必须避开 AI slop 默认词；用了必须解释为什么不能换
- 每个美学决策必须给理由
- brownfield 必走 1.5，观察报告必须经用户确认
- v0 草稿必须给用户确认（除非极小改动且用户明说"别 v0"）
- 禁止编造数据/好评/logo；缺素材走占位符策略

## Validation

- [ ] 4 个问题都有明确回答
- [ ] 调性是清单里的具体一项
- [ ] brownfield 走了 1.5 视觉语汇对齐
- [ ] v0 草稿给用户看了，已收到反馈
- [ ] 字体已显式避开 AI slop 默认
- [ ] 颜色用 OKLCH
- [ ] 给出可直接落地的 design tokens
- [ ] 关键组件规约 >= 5 类
- [ ] 占位符策略段已填
- [ ] anti-patterns 自检无命中（或命中已解释）

## Resources

- `references/UI-DESIGN.md` — 产出模板（含 frontmatter design tokens）
- `references/ui-aesthetics.md` — 决策框架（4 问 + 5 维度 + 调性卡片）
- `references/ui-anti-patterns.md` — 反 AI-slop 清单（8 类强制禁忌）
