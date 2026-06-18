---
name: draft-spec
description: >
  从 spec/detailed-requirements.md 产出一份方案（spec/PRD.md + spec/SPEC.md）。
  调 prd-writer + spec-writer agent 写出工业级文档。
  TRIGGER: /draft-spec
tools: Read, Write, Edit
---

# /draft-spec

## 前置条件

`spec/detailed-requirements.md` 必须存在（由 `/detail` 产出）。

## 概要流程

```
1. 读 detailed-requirements.md → 了解需求
2. 技术假设确认 → 对齐技术方向
3. 调 prd-writer agent → 写 spec/PRD.md（FR 格式工业级）
4. 调 spec-writer agent → 写 spec/SPEC.md（追溯+ADR）
5. PRD-SPEC 逐条核验
6. 输出确认
```

## 执行流程

### 第 1 步: 读输入

读以下文件：
1. `spec/detailed-requirements.md` — 已展开颗粒度的需求
2. `spec/SPEC.md` — 已有方案（更新场景）
3. 项目结构（给 agent 做上下文参考）

### 第 2 步: 技术假设确认（保留原版交互）

在写方案之前，先列出基于需求做出的关键假设，等你确认。原版交互，**给选项**：

```
基于需求，关于技术方向的几个假设：

1. 运行方式？
  A. 桌面端本地运行
  B. Web 服务端部署
  C. CLI 命令行工具

2. 数据存哪？
  A. 本地文件/SQLite
  B. 远程数据库（需联网）
  C. 不存

3. 用户模式？
  A. 单用户
  B. 多用户
  C. 匿名

以上假设对吗？确认后开始写方案。
```

每项确认或修正后，锁定假设再往下走。

### 第 3 步: 调 prd-writer（写 PRD）

通过 Agent tool 调用 `prd-writer`：

```
Agent tool:
  subagent_type: prd-writer
  prompt: >
    需求在 spec/detailed-requirements.md，技术假设已确认：
    [把第 2 步锁定的假设贴在这里]
    
    根据需求和技术假设，产出完整 PRD。
    格式: FR 编号 + GEARS 精确模式 + @TAG 追溯标记
    AC: Given/When/Then，覆盖 happy path + 边界 + 异常
    状态穷举、错误穷举、有兜底 E-999。
    输出到 spec/PRD.md。
```

prd-writer 执行完后，读 `spec/PRD.md` 确认已生成。

### 第 4 步: 调 spec-writer（写 SPEC）

通过 Agent tool 调用 `spec-writer`：

```
Agent tool:
  subagent_type: spec-writer
  prompt: >
    PRD 在 spec/PRD.md，项目结构在 [路径]。
    
    根据 PRD 和项目结构产出完整 SPEC。
    必须包含:
    - 需求追溯表（FR → SPEC 位置）
    - 技术栈对比（选什么 + 不选什么 + 理由）
    - 数据模型（DDL + 查询模式表 + < > 标注）
    - 接口契约（请求/响应/错误码穷举/限流）
    - 核心流程（Mermaid 图 + 文字 + 失败处理）
    - ADR（有取舍的决策）
    
    输出到 spec/SPEC.md。
```

spec-writer 执行完后，读 `spec/SPEC.md` 确认已生成。

### 第 5 步: PRD-SPEC 逐条核验

从 `spec/PRD.md` 读取 FR 清单，逐条检查 `spec/SPEC.md` 是否覆盖：

```
□ FR-001: [需求] → SPEC 中对应: [章节]   ✅ 已覆盖
□ FR-002: [需求] → SPEC 中对应: [章节]   ✅ 已覆盖
□ FR-003: [需求] → SPEC 中对应: [章节]   ⚠️ 部分覆盖（补充说明）
□ FR-004: [需求] → SPEC 中对应: [章节]   ❌ 未覆盖

全部覆盖 → 写入 SPEC
部分覆盖/未覆盖 → 补充后再写入
```

核验结果附在 `spec/SPEC.md` 末尾作为附录。

### 第 6 步: 输出确认

```
✅ 方案完成

📄 spec/PRD.md — N 条 FR（须: N, 应: N, 可: N, 不: N）
📄 spec/SPEC.md — 含追溯表 + N 个 ADR

质量摘要:
  FR 精确格式:      ✅ 全部 GEARS 模式
  验收标准:          ✅ 全部 Given/When/Then
  状态/错误穷举:     ✅ 有 E-999 兜底
  技术选型对比:      ✅ 每项都有"不选什么"
  需求追溯:          ✅ X/X 全部覆盖

建议下一步:
  /design-review "审查方案"    全量深审 → 修缺口
```

## 回退策略

如果 Agent tool 调用失败，回退到纯 Claude 写方案模式：
1. 读 detailed-requirements → 技术假设确认
2. 按 FR 格式手动写 PRD.md
3. 按 SPEC 模板手动写 SPEC.md
4. 核验 → 输出

## 强制产出章节

PRD.md 必须包含：
- FR 编号 + 须/应/可/不 分级
- GEARS 精确模式（事件式/状态式/条件式/恒定式/禁止式）
- Given/When/Then AC
- 状态穷举 + 错误穷举 + E-999 兜底
- @TAG 追溯标记

SPEC.md 必须包含：
- 需求追溯表（FR → SPEC 位置）
- 技术栈对比（选 + 不选 + 理由）
- 数据模型（DDL + 查询模式）
- 接口契约（请求/响应/错误码/限流）
- 核心流程（图 + 文字 + 失败处理）
- ADR（有取舍的决策）
