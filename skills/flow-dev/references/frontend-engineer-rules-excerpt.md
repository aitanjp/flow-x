# Frontend Engineer Rules — 节选

> 完整版见 `reference/frontend-engineer-rules.md`（~300 行）。本节选仅含 4-dev 阶段必跑的 第 1+2+10 节 + 按需节索引。

---

## 1. React + Babel 三条硬规则（不可协商）

### 1.1 禁止用 `const styles = {...}` 作全局变量名
**问题**：多个 `<script type="text/babel">` 文件里声明 `const styles` → 编译后同一作用域下互相静默覆盖。
**正确**：用组件名做命名空间：`const terminalStyles = { ... }`；或直接用内联 `style={{...}}`。

### 1.2 跨文件组件必须 `Object.assign(window, {...})`
**问题**：独立的 `<script type="text/babel">` 块作用域不共享。在 `Header.jsx` 里定义的组件在 `App.jsx` 里直接用会报 `Header is not defined`。
**正确**：组件定义文件尾部显式挂 window：`Object.assign(window, { Terminal, Line });`

### 1.3 禁用 `scrollIntoView`
**问题**：iframe 嵌入的预览环境里会触发外层 frame 滚动，造成视觉错乱。
**正确**：用 `element.scrollTop = ...` 或 `window.scrollTo({ top: ..., behavior: 'smooth' })`。

### 1.4 React CDN 版本必须固定
```html
<script src="https://unpkg.com/react@18.3.1/umd/react.development.js"
        integrity="sha384-hD6/rw4ppMLGNu3tX5cjIb+uRZ7UkRJ6BPkLpg4hAu/6onKUg4lLsHAs9EBPT82L"
        crossorigin="anonymous"></script>
```
禁止在 React CDN `<script>` 标签上加 `type="module"`。

---

## 2. CSS 硬规则

### 2.1 颜色一律来自 UI-DESIGN.md
- 颜色必须 OKLCH（遗留 hex 必须显式声明例外）
- 组件代码里**禁止**硬编码颜色字面值（`#fff` / `rgb(...)` / 原生色名）
- 所有颜色通过 CSS custom properties：`var(--color-brand)`

### 2.2 布局优先级
- CSS Grid + Flexbox 做主布局（不用 float / position: absolute 做布局）
- `text-wrap: pretty` 处理段落折行
- `clamp(min, fluid, max)` 做流体字号
- `@container` queries 做组件级响应

### 2.3 主题适配
- `@media (prefers-color-scheme)` 处理暗色
- `@media (prefers-reduced-motion)` 处理动效降级（任何 > 200ms 的动画都要 disable）

### 2.4 字体硬禁
默认禁止直接引用：
- 禁止 Inter / Roboto / Arial / Helvetica / system-ui 作主字体
- 禁止 Space Grotesk 作 display
- 禁止 "modern" / "clean" / "minimal" 作为调性描述词（必须给出具体名称）

---

## 10. 交付清单（提交前必须逐项过）

- [ ] 控制台无报错（React 严格模式 console.error 已清理）
- [ ] 所有状态有初始化（没有 `undefined` 作为初始值后未处理）
- [ ] 事件处理有清理（useEffect 返回清理函数，未清理的 setTimeout/setInterval 已处理）
- [ ] 组件有默认 props 或类型守卫
- [ ] 无硬编码颜色/字号（全部来自 design tokens）
- [ ] 无 `const styles` 全局变量
- [ ] 无 `scrollIntoView`
- [ ] 跨文件组件已挂 window
- [ ] 动画支持 prefers-reduced-motion
- [ ] 表单 label 显式关联（不靠 placeholder）
- [ ] 图片有 alt（装饰图用 `alt=""`）
- [ ] 焦点环可见（不是默认蓝色 outline）
- [ ] 键盘可达（Tab 顺序合理）

---

## 按需节索引

| 任务类型 | 额外阅读 |
|---|---|
| 交互原型 | 第 6.1 节 |
| 幻灯片/演示 | 第 6.2 节 |
| 仪表盘/数据可视化 | 第 6.3 节 |
| 动画/交互动效 | 第 4 节 + 第 6.4 节 |
| 多变体/实时调参 | 第 5 节（Tweaks 面板）|
