---
name: orchestrate
description: >
  7 步连续管线。从 /brief 开始（认知对话锚定需求），经过 /detail（需求详化）→ draft-spec → design-review → decompose → plan-task，最终到 /implement。
  用法：/orchestrate "写一个文件同步 CLI"
  TRIGGER: /orchestrate
tools: Skill
---

# /orchestrate

## 是什么

7 步连续管线。一句话需求 → 认知对话锚定 → 需求详化 → 技术方案 → 审查 → 拆分 → 计划 → 代码。

`/brief` 锚定"要做什么"，`/detail` 展开到可写方案的颗粒度，`/draft-spec` 才开始写方案。

## 管线

```
Step 0  /brief "一句话需求"      → spec/confirmed-requirements.md    (原子需求清单)
Step 0.5  /detail               → spec/detailed-requirements.md     (颗粒度展开)
Step 1  /draft-spec             → spec/PRD.md + spec/SPEC.md       (调 agent 写方案)
Step 2  /design-review "审方案"  → .scratch/审查报告    (4 维深审)
        你修缺口 ↑
Step 3  /decompose              → spec/phases.yaml     (任务拆分)
Step 4  /design-review "逐段审"  → 逐阶段审查报告
Step 5  /plan-task              → plans/tasks/*.plan.md (施工计划)
        你审批 ↑
Step 6  /implement              → 代码                 (逐 Phase 实现)
```

## 原型模式 `--proto`

```
Step 0  /brief "一句话需求"      → spec/PRD.md
Step 1  直接写原型代码           → 验证想法
```

原型跑通后，用完整模式走全流程做正式版。

## 上下文管理规则

**每步结束后建议新开会话**——管线是产物驱动的，不是对话驱动的。

```
Step N 完成 → 产物文件已写入磁盘
                ↓
          方案 A: 直接继续（轻量步骤，如 brief → draft-spec）
          方案 B: 新开会话，从产物冷启动（重量步骤，如 implement 每个 Phase）
```

- Step 0-1 (brief → draft-spec)：可串行，上下文消耗小
- Step 2-4 (design-review → decompose → 逐阶段审查)：每步后可新开会话
- Step 5-6 (plan-task → implement)：**强制每 Phase 新开会话**

## 执行逻辑

1. 检查当前项目目录，确定管线推进到哪了
   - 无 `spec/PRD.md` → 从 Step 0 开始 (/brief)
   - 有 PRD 无 SPEC → 从 Step 1 开始
   - 以此类推，自动断点续跑
2. 执行当前步骤，产出门禁文件
3. 每步完成后打印：
   - ✅ 刚完成什么
   - 📄 产物文件路径
   - ⏭ 下一步要做什么、你要确认什么
4. 关键门禁（Step 2 审查后修缺口、Step 5 plan 审批）停住等你

## 命令

- `/orchestrate "做一个文件同步工具"` — 完整 6 步
- `/orchestrate --proto "..."` — 原型模式，brief 后直接写代码
- `/orchestrate --status` — 显示当前进度
- `/orchestrate --step 2` — 直接从第 2 步开始（前序产物必须已存在）
- `/orchestrate --stats` — 遥测：管线历史通过率 + 卡点分析

## 遥测规则

**每步完成后，记录到 SQLite：**

```bash
bash ~/.claude/telemetry/pipeline-tracker.sh record \
  "orchestrate" \
  "<step-name>" \
  "<PASS|FAIL|SKIP>" \
  "<brief detail>" \
  "<duration_ms>"
```

- Step 名称映射: brief, draft-spec, design-review-spec, decompose, design-review-phases, plan-task, implement
- 不阻塞——record 失败只打到 stderr，不影响管线
- `/orchestrate --stats` 时调 `bash ~/.claude/telemetry/pipeline-tracker.sh stats` 展示汇总
