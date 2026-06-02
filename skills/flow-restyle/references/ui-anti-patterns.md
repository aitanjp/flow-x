# UI 反模式清单（精简版）

> 来源：Anthropic `frontend-design` skill + impeccable 项目 anti-pattern 库。

## 强制禁忌（命中即必须改）

### 字体类
- ❌ **Inter / Roboto / Arial / Helvetica / system-ui** 作主字体（除非品牌强制）
- ❌ Space Grotesk 用作 display（已被用烂）
- ❌ Display 和 body 是同一个字体的不同字重

### 颜色类
- ❌ **纯黑 `#000` / 纯白 `#fff`**（用倾斜中性色）
- ❌ **紫色渐变 on 白底**（最强 AI 标志）
- ❌ **霓虹青 on 黑底**（AI 工具营销页常见）
- ❌ **两个或更多强调色**
- ❌ Gradient text（`background-clip: text` + 渐变）已 cliché

### 阴影类
- ❌ 静态卡片带 drop shadow（at rest 应该是平的）
- ❌ Shadow alpha > 0.15
- ❌ 装饰性彩色阴影
- ❌ Inset shadow + glow 同时用（油腻）

### 边框类
- ❌ **`border-left` 或 `border-right` > 1px 作彩色侧条**（最强 AI dashboard 标志）
- ❌ 渐变边框作装饰
- ❌ 玻璃拟态（glassmorphism）

### 动效类
- ❌ **Bounce / elastic 缓动**
- ❌ animate `width` / `height` / `padding` / `margin`
- ❌ 散落的微交互（不如一次编排好的入场动画）
- ❌ 不支持 `prefers-reduced-motion`
- ❌ 滚动劫持（scroll hijacking）

### 布局类
- ❌ **卡片嵌套卡片**
- ❌ **统一大小卡片网格 + 图标 + 标题 + 文本 + 重复 N 次**（SaaS template）
- ❌ **Hero 大数字 + 小标签 + 副指标 + 渐变装饰**
- ❌ 统一 4/8/12/16/20 间距（用非线性 scale）
- ❌ 默认 dark mode（除非产品就是夜间使用）

### 文案类
- ❌ "Boost your productivity" / "Unleash your potential" 类空话
- ❌ Lorem ipsum 出现在最终交付里
- ❌ 按钮写 "Submit" / "Click here" / "Learn more"

### 组件类
- ❌ 圆角矩形 + 通用 drop shadow 的 button
- ❌ 在 form 里用 placeholder 替代 label
- ❌ Tooltip 在移动端依赖 hover
- ❌ 模态框无 escape 键关闭
- ❌ Skeleton loader 用纯灰条

## 判断模糊地带

| 模式 | 何时可用 | 何时禁止 |
|---|---|---|
| 默认字体 (Inter 等) | 行政内部工具 | 任何 marketing / brand 页 |
| 紫色 | 项目本来就是紫色品牌 | 通用应用为了"高级感"加紫色渐变 |
| Drop shadow | hover/focus 状态响应 | 静止卡片装饰 |
| 第二个强调色 | 语义需要（成功/警告/危险） | 视觉装饰 |
| Dark mode | 阅读 / 视频 / 编程工具 | 编辑式 / marketing |
| Glassmorphism | 极少数"玻璃 HUD"特效 | 任何 marketing 页 |
