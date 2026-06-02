# UI 反模式清单

> 来源：Anthropic `frontend-design` skill + impeccable 项目 anti-pattern 库。

---

## 强制禁忌（命中即必须改）

### 字体类
- 禁止 Inter / Roboto / Arial / Helvetica / system-ui 作主字体（除非品牌强制）
- 禁止 Space Grotesk 用作 display
- 禁止 Display 和 body 是同一个字体的不同字重

### 颜色类
- 禁止纯黑 #000 / 纯白 #fff
- 禁止紫色渐变 on 白底
- 禁止霓虹青 on 黑底
- 禁止彩色背景上配灰色文字
- 禁止两个或更多强调色
- 禁止 Gradient text（background-clip: text + 渐变）

### 阴影类
- 禁止静态卡片带 drop shadow
- 禁止 Shadow alpha > 0.15
- 禁止装饰性彩色阴影
- 禁止 Inset shadow + glow 同时用

### 边框类
- 禁止 border-left 或 border-right > 1px 作彩色侧条
- 禁止渐变边框作装饰
- 禁止玻璃拟态（glassmorphism）

### 动效类
- 禁止 Bounce / elastic 缓动
- 禁止 animate width / height / padding / margin
- 禁止散落的微交互
- 禁止不支持 prefers-reduced-motion
- 禁止滚动劫持

### 布局类
- 禁止卡片嵌套卡片
- 禁止统一大小卡片网格 + 图标 + 标题 + 文本重复
- 禁止 Hero 大数字 + 小标签 + 副指标 + 渐变装饰
- 禁止统一 4/8/12/16/20 间距
- 禁止默认 dark mode（除非产品就是夜间使用）

### 文案类
- 禁止 "Boost your productivity" / "Unleash your potential" 类空话
- 禁止 "Maybe consider..." / "It might be helpful..."
- 禁止 Lorem ipsum 出现在最终交付
- 禁止按钮写 "Submit" / "Click here" / "Learn more"

### 组件类
- 禁止圆角矩形 + 通用 drop shadow 的 button
- 禁止在 form 里用 placeholder 替代 label
- 禁止 Tooltip 在移动端依赖 hover
- 禁止模态框无 escape 键关闭
- 禁止 Skeleton loader 用纯灰条

---

## 判断模糊地带

| 模式 | 何时可用 | 何时禁止 |
|---|---|---|
| 默认字体 (Inter 等) | 行政内部工具 | 任何 marketing / brand 页 |
| 紫色 | 项目本来就是紫色品牌 | 通用应用为了"高级感" |
| Drop shadow | hover/focus 状态 | 静止卡片装饰 |
| 第二个强调色 | 语义需要（成功/警告/危险） | 视觉装饰 |
| Dark mode | 阅读/视频/编程工具 | 编辑式 / marketing |
| Glassmorphism | 极少数玻璃 HUD 特效 | 任何 marketing 页 |

---

## 给 AI 的扫描指令

UI 任务进入实现前：
1. 用关键词 grep 本文件相关章节
2. 把命中的禁忌逐条声明
3. 实现完成后再扫一遍

UI review 阶段：
1. 把 diff 与本清单逐条比对
2. 命中即标 Critical，追加 fix 任务
