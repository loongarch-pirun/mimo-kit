---
name: refactor-execute
description: >
  重构阶段3 — 按清单执行。读 tasks.yaml，逐条调度 agent 修代码，测试验证，checkpoint 断点续跑。
  TRIGGER: /refactor-execute
---

# /refactor-execute

## 用法

```
/refactor-execute <tasks.yaml 路径>
```

## 前置条件
- tasks.yaml 已存在（由 `/refactor-diagnose` 产出）
- `spec/PRD.md` 和 `spec/SPEC.md` 存在（可选，有的话修得更准）

## 核心流程

```
1. 读 tasks.yaml → 了解全部任务
2. 读 checkpoint（如果有）→ 知道修到哪了
3. 读 PRD.md + SPEC.md（如果有）→ 了解需求和方案
4. 循环处理任务（逐条，不并行）：
     取一条未完成的任务 →
     读取 assign 字段确定用哪个 agent →
       ↓
     ┌────────────────────────────────────────────┐
     │          每条任务的完整内循环                │
     │                                            │
     │  第1步：修正/重构（修）                      │
     │    调对应的 agent                       │
     │    传给 agent：在哪改 + 改什么 + 需求       │
     │              + 方案 + 操作指引 + 验证标准    │
     │    agent 改完代码 → 输出 diff               │
     │                                            │
     │  第2步：diff 审查（审）                      │
     │    调 code-reviewer 或另一个独立 agent      │
     │    查：改了哪些文件？只改了该改的吗？        │
     │    查：有没有引入新问题？有没有破坏相邻代码？ │
     │    查：改动跟 action 和 verify 对得上吗？   │
     │                                            │
     │  第3步：测试验证（测）                       │
     │    跑项目测试套件                           │
     │    跑 verify 字段定义的专项验证              │
     │    （安全扫描 / lint / 类型检查 / grep 等）  │
     │                                            │
     │  第4步：通过判定（过）                       │
     │    diff 审查通过 + 测试通过 + 验证通过       │
     │      → 更新 checkpoint → 取下一条           │
     │    任一不通过 → 回溯重试（最多 2 次）        │
     │    仍不通过 → 标记 BLOCKED，记录原因 → 下一条 │
     └────────────────────────────────────────────┘
5. 全部完成或碰到阻塞 → 打印汇总
```

### 内循环说明

**第1步（修）**：调 `assign` 指定的 agent。重点是让它带着上下文去改，不改范围之外的东西。

**第2步（审）**：修代码的人不能审自己的代码。调 code-reviewer 或其他独立 agent 做 diff 审查。只查 diff，不查全部代码。

**第3步（测）**：先跑项目已有测试套件（确保不改坏东西），再跑 `verify` 定义的专项验证。

**第4步（过）**：三步全通过才算过。有一项不通过就回溯重试，最多 2 次。

## 任务分派说明

tasks.yaml 中每条任务的 `assign` 字段直接决定用哪个 agent。配对的逻辑不是"名字对上就行"，而是**按问题类型精准路由到最专业的 agent**。

### 精准配对原则

```
安全漏洞（SQL注入/密钥/XSS）       → security-reviewer
  因为 security-reviewer 是唯一专门扫安全的 agent，其他 agent 不懂 OWASP

代码简化（长函数/重复/复杂逻辑）     → code-simplifier
  因为 code-simplifier 的专长就是简化，不会顺手改业务逻辑

错误处理（空catch/吞异常/无日志）    → silent-failure-hunter
  因为它只查"异常有没有被正确处理"，不碰其他

Python 代码重写                    → python-reviewer
TypeScript 代码重写                → typescript-reviewer
React 组件重写                     → react-reviewer
Go 代码重写                        → go-reviewer
Rust 代码重写                      → rust-reviewer
Java 代码重写                      → java-reviewer
C++ 代码重写                       → cpp-reviewer
  以上全是语言/框架专项 agent，比通用 agent 懂的更多

测试覆盖不足                       → tdd-guide
  因为 tdd-guide 是所有 agent 里唯一强制先写测试再写代码的

死代码清理                         → refactor-cleaner
  因为它会先跑工具（knip/depcheck）确定没在用，再删，不靠猜

性能优化（慢查询/大循环/内存泄漏）   → performance-optimizer
  因为它会先分析瓶颈在哪，再动代码，不盲目优化

架构偏离                           → architect
  因为 architect 是最懂"方案里怎么设计"的

通用代码质量                       → code-reviewer
  上面没有精确匹配的 fallback
```

### 调用方式

通过 Agent tool 调用，`subagent_type` 参数设为 `assign` 的值：

```
Agent tool:
  subagent_type: security-reviewer   ← 从 task.assign 读取
  prompt: 包含 problem + context + action + verify
```

不在上表中的 `assign` 值，直接用 `subagent_type: <assign 值>` 调用。Agent tool 会自动去 `~/.claude/agents/<name>.md` 找对应的 agent 定义文件。

## 验证方式

每条任务验证方式由 `verify` 字段定义。常见验证方式：

```
- 跑测试:  运行项目测试套件
- 安全扫描: 运行 trivy/gitleaks
- 类型检查: 运行 tsc --noEmit
- lint:    运行 eslint
- grep 确认: 搜索特定模式确保已修复
```

验证未通过 → 回溯重试（最多 2 次）。仍不通过 → 标记 `BLOCKED`，记录原因，跳过。

## 上下文管理

如果任务列表超过 10 条，分批处理（每批 5 条）：

```
取 5 条 → 修完 → 写 checkpoint → 看上下文剩余
  够用 → 继续下 5 条
  不够 → 打印"建议压缩上下文后继续 /refactor-execute"
```

每批完成后写入 checkpoint：

```json
{
  "tasks.yaml": ".scratch/refactor-tasks/tasks.yaml",
  "total": 15,
  "done": 5,
  "blocked": 0,
  "pending": 10,
  "current_batch": 2,
  "last_task": "R005",
  "status": "in_progress"
}
```

下次 `/refactor-execute` 时自动从 checkpoint 继续。

## 产出

```
全部任务完成后打印：

✅ 重构执行完成
  总任务: N 条
  已完成: N 条
  已阻塞: N 条（需人工确认）
  清单路径: .scratch/refactor-tasks/tasks.yaml

  建议下一步: 压缩上下文 → 开新会话 → 跑四 Agent 体检
  （cognitive-map + convolution-audit + stress-test + repair-engine）
```
