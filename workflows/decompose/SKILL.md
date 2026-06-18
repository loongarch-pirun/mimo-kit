---
name: decompose
description: >
  把 SPEC 拆成 spec/phases.yaml — Phase 1/2/3 依赖图。
  每任务带 ID、验收标准、依赖关系、估时、类型。
  TRIGGER: /decompose
tools: Read, Write, Edit
---

# /decompose

## 前置条件

`spec/SPEC.md` 必须存在（由 `/draft-spec` 产出）。

## 算法（主 Claude 直接执行，不委派 subagent）

1. **识别最小端到端可工作切片** → Phase 1
   - Phase 1 目标：骨架端到端跑通（用 mock/种子数据即可）
2. **识别核心功能闭环** → Phase 2
   - Phase 2 目标：真实用户能完成核心 Job-To-Be-Done
3. **识别打磨 + 上线预备** → Phase 3
   - Phase 3 目标：达到公开发布的质量门槛

## 每任务字段

- `id: P{phase}-T{nn}` — 唯一标识
- `title:` 动词 + 名词，最多 8 词
- `description:` 1 段话，说明任务做什么
- `type:` `foundation | data | backend | frontend | design | infra | content | integration`
- `priority:` `P0 | P1 | P2`
- `depends_on: []` — 前置任务 ID 列表
- `estimate:` `S | M | L | XL`
- `acceptance:` 具体的、可验证的验收标准（1-3 条）

## 硬性规则

- ❌ 不允许 XL 任务直接执行 — 必须拆
- ✅ 每个 Phase 有显式的 `gate_criteria`（该 Phase 全部完成的验收标准）
- ✅ 依赖图必须是无环图（DAG）— 手动检查循环依赖

## 精细化

展示 YAML 给用户，请用户做一轮审查。然后将 YAML 写入 `spec/phases.yaml`。

打印：

```
phases.yaml 已写入 → spec/phases.yaml

✓ 任务已分解。下一步:
  /design-review "逐阶段审查"    逐阶段深审 → 确保每个阶段可执行
  /plan-task                     进入单阶段计划
```
