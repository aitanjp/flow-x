# REVIEW: <change 标题>

- **Change ID**: <id>
- **审查时间**: <YYYY-MM-DD HH:mm>
- **审查者**: AI（Reviewer 角色）+ 跨模型 spot-check（如有）
- **总体结论**: 通过 / 需修复 / 阻塞

---

## 第一轮 · Spec 合规审查

| 检查项 | 结果 | 证据 |
|---|---|---|
| 每条 AC 都已实现 | | |
| 每条 AC 都有测试 | | |
| 未引入 `out of scope` 内容 | | |
| 未范围蔓延 | | |
| 未越过 DESIGN 边界 | | |

**Spec 合规结论**: 

---

## 第二轮 · 代码质量审查（6 维衰退风险）

### 2.0 TEST.md 5 轮金字塔完整性

| 轮次 | 状态 | 缺漏 |
|---|---|---|
| 1 功能 | | |
| 2 性能 | | |
| 3 安全 | | |
| 4 兼容 | | |
| 5 可观测 | | |

### 2.1 6 维诊断 · 严重度统计

| 编号 | 衰退风险 | 🔴 | 🟡 | 🟢 |
|---|---|---|---|---|
| R1 | Cognitive Overload | | | |
| R2 | Change Propagation | | | |
| R3 | Knowledge Duplication | | | |
| R4 | Accidental Complexity | | | |
| R5 | Dependency Disorder | | | |
| R6 | Domain Model Distortion | | | |

### 2.2 6 维诊断 · 详细发现（4 要素）

```markdown
### 🔴/🟡/🟢 R<x> · <名字>：<结论>
**Symptom**：<file:line>
**Source**：<书名 · 章节>
**Consequence**：<不修会怎样>
**Remedy**：<具体怎么改>
**生成 fix 任务**：T-FIX-NN
```

### 2.3 架构依赖图（大型 change）

```mermaid
<贴 brooks-audit 输出 / 内置简化图>
```

**循环依赖**：
**反向依赖**：

---

## 第三轮 · UI 视觉审查（仅前端）

### 3.1 Design Tokens 一致性

- [ ] 颜色全部来自 UI-DESIGN.md frontmatter
- [ ] 无硬编码 hex/字号/间距
- [ ] 字体一致

### 3.2 Anti-Pattern 扫描

- [ ] 字体类
- [ ] 颜色类
- [ ] 阴影类
- [ ] 边框类
- [ ] 动效类
- [ ] 布局类
- [ ] 文案类
- [ ] 组件类

### 3.3 视觉北极星一致性

### 3.4 无障碍快检

- [ ] 颜色对比 >= WCAG 2.1 AA
- [ ] 键盘可达
- [ ] 焦点环可见
- [ ] prefers-reduced-motion
- [ ] 表单 label
- [ ] 图片 alt

---

## 第四轮 · 补充审查（按触发条件）

### 4.1 技术债评估

### 4.2 跨模型分歧

| 主审发现 | 跨模型发现 | 是否一致 | 处理 |
|---|---|---|---|
| | | | |

---

## 总结

- Critical 项：<数量>，已全部生成 fix 任务
- Major 项：<数量>，已修：<>，已知接受：<>（需人工签字）
- Minor 项：<数量>，建议下次顺手优化

**下一步**: 进入 flow-dev 修复 Critical / 进入 flow-integration
