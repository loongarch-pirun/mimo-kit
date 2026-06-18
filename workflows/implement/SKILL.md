---
name: implement
description: >
  按审批过的 plan + phases.yaml 逐 Phase 写代码。
  每 Phase 跑测试 + convolution-audit + stress-test 门禁。
  Refuses without approved plan.
  TRIGGER: /implement
tools: Edit, Read, Write, Bash(*)
---

# /implement

## 前置条件

- `plans/tasks/*.plan.md` 存在（由 `/plan-task` 产出）
- Plan 已标注可执行的 Phase
- 用户已审批

## 执行循环

按 Phase 顺序执行，每个 Phase 内部：

1. 读该 Phase 下所有 task 的 plan
2. 按依赖顺序（`depends_on`）写代码 — 每任务调用 tdd-guide agent：先写测试(RED) → 确认失败 → 写代码(GREEN) → 确认通过 → 验证覆盖率 ≥ 80%
3. 每任务完成后立刻跑 SpecGuard 验证：
   ```
   Task 完成 →
     ├── spec_check_interfaces (type=backend 时)
     ├── spec_check_models    (type=data 时)
     ├── spec_check_security  (type=backend 时)
     ├── spec_check_env       (type=infra 时)
     ├── PASS → 继续下一个 task
     ├── WARN → 记录偏差，继续（不阻塞）
     └── FAIL → 停，显示偏差报告，修完再继续
   ```
4. Phase 全部任务完成后 → 轻量门禁（无外部依赖）：

```
Phase N 代码写完 →
  ├── build check (项目能编译吗)
  ├── type check (类型检查通过吗)
  ├── lint check (代码规范通过吗)
  ├── test suite (测试全部通过 + 覆盖率 80%+)
  ├── security scan (grep 扫描泄露的 key / console.log)
  ├── diff review (改了哪些文件，有没有意外改动)
  ├── ✅ 通过 → 继续 Phase N+1
  └── ❌ 不通过 → 修复 → 重跑 Phase N
```
5. 全部 Phase 完成后 → 启动四 Agent 全面体检：
   ```
   implement 全部 Phase 完成 →
     ├── cognitive-map Agent    → 全景代码库测绘 (体检报告 + 交接单)
     ├── convolution-audit Agent → 硬证据安全审计 (trivy + gitleaks 前置扫描)
     ├── stress-test Agent      → 4 阶段极限压测 (多 agent 投票审查)
     ├── repair-engine Agent    → 有 FAIL 时自动修复 → 收敛至门禁通过
     └── 输出完整体检报告
   ```

## 断点续跑

每 Phase 完成写 `.scratch/checkpoint.json` 并做**上下文切段**：
- 当前 Phase 的所有 task 代码、测试结果、门禁结果已写入文件
- 下一个 Phase 从 checkpoint + SPEC + plan **冷启动**（不带当前对话历史）
- 写完 checkpoint 后主动提示: "Phase N 完成，建议新开会话执行 Phase N+1"

```json
{
  "plan": "spec/phases.yaml",
  "total_phases": 3,
  "completed_phases": [1],
  "current_phase": 2,
  "last": "2026-06-12T14:30:00Z"
}
```

下次 `/implement` 自动从断点继续。

## 硬性规则

- 先读 plan 再写代码
- 每 task 完成跑 SpecGuard 对照 SPEC
- SpecGuard FAIL 的任务不得标记为完成，修好再进下一个 task
- 每 Phase 跑门禁，不跳过
- 只写当前 Phase 的代码，不提前写后面的
- 偏离 plan 超过 30%（改动文件数超出预测）→ 立刻停，更新 plan 再继续

## 输出

Phase N 完成时打印：
- 改了哪些文件
- 测试结果
- 门禁结果
- 能否进下一 Phase
