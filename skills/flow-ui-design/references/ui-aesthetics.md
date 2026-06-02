# UI 美学决策框架

> 融合自 Anthropic 原版 `frontend-design` skill + 社区 `impeccable` 项目精华。

---

## 决策的 4 个问题（开工前必须想清楚）

### 1. 目的（Purpose）
- 这个界面解决什么问题？
- 用户在什么场景下打开它？
- 用户期待的核心动作是什么？

### 2. 调性（Tone）
选一个明确的调性方向。9 种主流方向见下方卡片。

### 3. 约束（Constraints）
- 技术栈 / 性能 / 无障碍 / 品牌限定

### 4. 差异化（Differentiation）
- 让人记住的"那一件事"是什么？
- 三天后用户描述这个产品会用什么形容词？

---

## 9 种主流调性方向

1. **编辑式（Editorial）**：杂志感 / 衬线显示字 / 大留白 / 暖纸色。参考：A List Apart、Apple Newsroom。适合：内容驱动 / 品牌权威 / 长文
2. **极简（Minimal）**：单色 / 几何字 / 无装饰 / 留白。参考：Linear、Vercel、Stripe。适合：技术工具 / B2B SaaS
3. **工业（Industrial）**：等宽字 / 暴露网格 / 冷色 / 数据密度。参考：Bloomberg、Grafana。适合：监控 / 金融 / 工程师工具
4. **玩具（Playful）**：圆角 / 高饱和 / 弹动 / 插画。参考：Duolingo、Headspace。适合：教育 / 儿童 / 协作
5. **奢华（Luxury）**：衬线 / 金属质感 / 深色 + 金 / 大留白。参考：Aesop、Bottega Veneta。适合：高端品牌 / 奢侈品
6. **有机（Organic）**：流动形 / 暖色 / 不规则 / 自然意象。参考：Headspace、Calm、Apple Health。适合：健康 / 冥想 / 食品
7. **复古未来（Retro-Futurism）**：80s/90s / 像素 / 霓虹。参考：Cyberpunk、Vercel Conf。适合：游戏 / 创意工具 / Web3
8. **粗野（Brutalist）**：单一字 / 黑白 / 网格暴露 / 反美学。参考：Balenciaga、设计师作品集。适合：设计圈 / 反主流 / 艺术
9. **极繁（Maximalist）**：信息密集 / 多层叠加 / 装饰主义。参考：Eye Magazine。适合：内容广场 / 节日主题

---

## 五大美学维度（实施层）

### 1. 字体（Typography）
- 选一对：display 字体 + body 字体
- **绝对避开** AI slop 默认：Inter / Roboto / Arial / system-ui
- 行高 body **1.6**（load-bearing 参数）
- 流体头部用 `clamp()`；body 不用流体

### 2. 颜色（Color）
- **用 OKLCH**，不用 hex
- **倾斜中性色**：给中性色一点点品牌色 hue（chroma 0.005~0.02）
- **一个声音原则**：主色调只有 1 个
- 避免 #000 / #fff，用 oklch(10% ...) / oklch(98% ...)

### 3. 动效（Motion）
- 缓动曲线默认 expo-out：`cubic-bezier(0.16, 1, 0.3, 1)`
- **禁止 bounce / elastic**
- 时长档位：颜色 150ms；transform 300-400ms；编排入场 600-1200ms
- 只动 transform 和 opacity
- 必须支持 `prefers-reduced-motion`

### 4. 空间布局（Spatial）
- 间距用非线性 scale：8/16/24/32/48/80/120
- 密度二选一：大留白（editorial）或受控密度（dashboard）

### 5. 背景与质感（Atmosphere）
- 考虑：渐变网格 / 噪点纹理 / 几何图案 / 分层透明度 / 戏剧性阴影

---

## 阴影词汇

- 组件 at rest 不带阴影
- 阴影只在 hover/focus/主动抬起时出现
- 每个阴影最强 blur 的 alpha <= 0.15
- 着色阴影只给"刻意的强调时刻"
