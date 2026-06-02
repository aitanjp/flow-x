# TASK: <change 标题>

- **Change ID**: <id>
- **关联**: `@.specs/<id>/REQUIREMENT.md`、`@.specs/<id>/DESIGN.md`

---

## 波次划分

```
Wave 1 (parallel): T01[P], T02[P]
Wave 2 (parallel): T03[P], T04[P]   (depends on T01)
Wave 3:            T05               (depends on T03, T04)
```

> 同 wave = 可并行；跨 wave = 必须顺序执行。

---

## 任务清单

```xml
<task id="T01" parallel="true" status="pending">
  <name><一句话任务名></name>
  <read_files>
    <参考边界 · 允许 read 的文件，支持 glob>
  </read_files>
  <write_files>
    <修改边界 · 严格控制，超出会被 R6.5 提交前 verify 拦住>
  </write_files>
  <action>
    <做什么。写意图，不写代码。>
  </action>
  <verify>
    <一条可执行验证命令>
  </verify>
  <done>
    <完成判定。一句话，对应 AC 的某个子项>
  </done>
  <depends_on></depends_on>
</task>
```

> **注意**：`read_files` 和 `write_files` 是 R7.3 强约束。`write_files` 必须严格在 DESIGN `## 0.5.1` 「触碰模块 + 新增模块」范围内，且不能包含「禁动清单」。

---

## 状态字段说明

- `status="pending"` — 未开始
- `status="in_progress"` — 进行中
- `status="done"` — 已完成（verify 通过）
- `status="blocked"` — 阻塞（必须在「阻塞日志」记录）

---

## 阻塞日志

| 任务 | 阻塞原因 | 待人工决策项 | 时间 |
|---|---|---|---|
|  |  |  |  |

---

## Fix 任务（来自 REVIEW / INTEGRATION）

> 此区域由 review/integration 阶段自动追加，编号 `T-FIX-XX`。

```xml
<!-- 占位 -->
```
