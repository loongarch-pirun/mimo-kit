---
name: plan-task
description: >
  对当前阶段的任务执行 Plan 模式 — 产出结构化施工计划。
  读取 phases.yaml + SPEC → 为当前 Phase 每任务生成 plan。
  TRIGGER: /plan-task
tools: Read, Write, Edit, Grep, Glob
---

# /plan-task

## 前置条件

`spec/phases.yaml` 必须存在（由 `/decompose` 产出）。

## 执行步骤

1. **读取 `spec/phases.yaml`** — 提取当前 Phase 的所有任务
2. **读取 `spec/SPEC.md`** — 获取技术上下文
3. **逐任务生成施工计划**，写入 `plans/tasks/{id}.plan.md`：

每份 plan 包含：

- **目标** — 这个任务要达成什么
- **涉及文件**（预测）— 根据 SPEC + 项目结构推断要改/新建哪些文件
- **数据流 / 组件树** — 该任务内部的关键数据路径或组件关系
- **关键边界条件** — 极端输入、并发、异常路径
- **测试策略** — 单元测试 / 集成测试 / E2E 分别在哪儿测
- **待确认问题** — 如果 SPEC 或 phases.yaml 没覆盖的细节，标记出来

4. **汇总展示** — 把该 Phase 所有任务的 plan 文件名和一句话目标列出来
5. **等待用户审批后继续** — 审批通过后 `/implement` 才被解锁

## 硬性规则

- 一个任务（P{phase}-T{nn}）对应一份 plan 文件
- 如果 SPEC 对某任务的信息不足以写出 plan → 标记 `[待确认]`，不编造
- Plan 文件格式禁止省略注释

## 输出

```
Phase N 施工计划已就绪:

| 文件 | 任务 |
|------|------|
| plans/tasks/P1-T01.plan.md | ... |
| plans/tasks/P1-T02.plan.md | ... |

✓ 审查以上 plan 后，用 /implement 开始写代码。
```
