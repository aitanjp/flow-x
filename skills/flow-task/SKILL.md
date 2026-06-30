---
name: flow-task
description: 任务拆解规划师。把 DESIGN.md（+ UI-DESIGN.md）拆成原子任务列表 TASK.md，含 XML 格式任务（7 字段：id/name/read_files/write_files/action/verify/done）、波次依赖图、[P] 并行标记和可执行 verify 命令。GitNexus MCP 可用时用于精确定位 write_files 边界和既有模块依赖。Use when 用户确认 DESIGN.md（前端项目还需确认 UI-DESIGN.md）后需要进入任务拆解阶段，或需要重新规划任务时。
---

# flow-task — 任务拆解

## Goal

把设计拆成可并行的原子任务，每个任务在 fresh context 下 2~10 分钟可完成，且自带可执行的 verify。

## Workflow

### 入口门禁（Artifact Preflight）

开始拆任务前检查上游工件：
- 缺 REQUIREMENT.md → 停止，回 flow-requirement
- 缺 DESIGN.md → 停止，回 flow-design
- 前端/UI 项目缺 UI-DESIGN.md → 停止，回 flow-ui-design
- 禁止 Planner 自己脑补技术栈、触碰模块、禁动清单或 write_files 边界

### 拆解原则

1. **大小**：一个任务在 fresh context 下 2~10 分钟可完成
2. **粒度**：按文件冲突切，不按层切。优先「垂直切片」（一个特性贯穿模型/API/UI）
3. **并行标记 `[P]`**：互不冲突的任务标 `[P]`，同波次并行执行
4. **依赖**：每个任务显式声明 `depends_on: <task-id>`
5. **每任务必备 7 字段**：
   - `id` —— 形如 `T01`、`T02-1`
   - `name` —— 一句话
   - `read_files` —— 参考边界（支持 glob）
   - `write_files` —— 修改边界（超出会被 R6.5 提交前 verify 拦住）
   - `action` —— 要做什么（写意图，不写代码）
   - `verify` —— 一条可执行的验证命令
   - `done` —— 完成判定（对应 AC 的某个子项）

### read_files 与 write_files 的区别

- **read_files 应该包含**：
  - 本任务要修改的文件（= write_files 的超集）
  - 本任务要 import / 参考的既有模块
  - DESIGN.md `## 0.5.1` 「触碰模块」中的「已有·复用」项

  **GitNexus 路径（可选）**：对任务 action 涉及的能力调用 `gitnexus_query(query="<action 关键概念>", goal="identify files to read for implementation")`，精确定位需要参考的既有模块文件。

- **write_files 严格控制**：
  - DESIGN.md `## 0.5.1` 「新增模块」项
  - DESIGN.md `## 0.5.1` 「触碰模块」中需要修改的那些
  - **不允许加「禁动清单」里的文件**

### 波次划分

把任务按依赖图分层：
- 同层 = 同波次（并行执行）
- 跨层 = 顺序执行

```
Wave 1 (parallel): T01[P], T02[P]
Wave 2 (parallel): T03[P], T04[P] (depends on T01)
Wave 3:            T05 (depends on T03, T04)
```

## Constraints

- **R2.3**：每个任务必须有可执行的 `verify`，否则不允许进入 DEV
- 任务粒度太大（无法在 fresh context 完成）必须再拆
- 不允许「重构 X 模块」这种没有边界的任务

## Validation

- [ ] 每个任务都有完整的 7 字段
- [ ] 每个 write_files 都严格在 DESIGN「触碰模块 + 新增模块」范围内
- [ ] 任何任务的 write_files 都不包含 DESIGN「禁动清单」中的文件
- [ ] 每个任务的 verify 都是可执行命令
- [ ] 至少有 1 个 `[P]` 标记的并行任务
- [ ] 波次划分图清晰、无环依赖
- [ ] 任务编号连续

## Resources

- `references/TASK.md` — 产出模板（含 XML schema + 波次示例）
