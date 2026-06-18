---
name: refactor-diagnose
description: >
  重构阶段2 — 串行诊断。检测项目语言，按需加载 agent。读PRD+SPEC，跑 N 个 agent 审查代码，合并为任务清单。
  TRIGGER: /refactor-diagnose
---

# /refactor-diagnose

## 用法

```
/refactor-diagnose <项目根目录路径>
```

项目根目录下需包含：
- `spec/PRD.md`（需求）
- `spec/SPEC.md`（技术方案）

如无 spec 目录也可直接指定，流程会自动适配。

## 流程

```
0. 检测项目使用的语言和框架（见下方"语言检测"）→ 确定本轮诊断 agent 阵容
1. 读 PRD.md + SPEC.md（了解需求与方案）
2. 检测项目特征，决定是否开启 performance-optimizer
3. 串行跑 N 个 agent，每人带着方案看代码：

     基础 agent（必选，始终运行）:
       architect             → 架构实现跟 SPEC 对不对得上
       code-reviewer         → 代码质量审查
       security-reviewer     → 安全漏洞扫描
       silent-failure-hunter → 错误处理缺陷扫描
       pr-test-analyzer      → 测试覆盖分析

     语言 agent（按检测结果追加，0 到多个）:
       python-reviewer       → Python 代码专项（检测到 .py）
       typescript-reviewer   → TypeScript 专项（检测到 .ts/.tsx）
       react-reviewer        → React 组件专项（检测到 .tsx/.jsx + react import）
       go-reviewer           → Go 专项（检测到 go.mod）
       rust-reviewer         → Rust 专项（检测到 Cargo.toml）
       java-reviewer         → Java/Spring 专项（检测到 pom.xml/.gradle）
       cpp-reviewer          → C++ 专项（检测到 .cpp/.hpp）
       csharp-reviewer       → C# 专项（检测到 .csproj）
       swift-reviewer        → Swift 专项（检测到 .swift）
       kotlin-reviewer       → Kotlin 专项（检测到 .kt/.gradle.kts）
       flutter-reviewer      → Flutter/Dart 专项（检测到 pubspec.yaml）
       django-reviewer       → Django 专项（检测到 manage.py + requirements.txt）
       fastapi-reviewer      → FastAPI 专项（检测到 fastapi import）

4. 合并所有发现 → 去重 → 定性 → 排优先级 → 指派 agent
5. 产出一份完整的 tasks.yaml

每次调用后打印串联摘要，方便用户决定是否压缩上下文继续。
```

## 语言检测

在项目根目录运行以下检测项，发现匹配则追加对应 agent 到诊断链：

| 检测条件 | agent | 追加时机 |
|----------|-------|---------|
| `find . -name "*.py" \| head -5` | python-reviewer | 有 .py 文件 |
| `find . -name "*.tsx" -o -name "*.ts" \| head -5` | typescript-reviewer | 有 TypeScript 文件 |
| `grep -r "react" package.json --max-count=1` | react-reviewer | package.json 里有 react |
| `ls go.mod 2>/dev/null` | go-reviewer | 存在 go.mod |
| `ls Cargo.toml 2>/dev/null` | rust-reviewer | 存在 Cargo.toml |
| `ls pom.xml 2>/dev/null -o ls build.gradle* 2>/dev/null` | java-reviewer | 存在 Maven/Gradle |
| `find . -name "*.csproj" \| head -3` | csharp-reviewer | 有 .csproj 文件 |
| `find . -name "*.swift" \| head -3` | swift-reviewer | 有 .swift 文件 |
| `find . -name "*.kt" \| head -3` | kotlin-reviewer | 有 Kotlin 文件 |
| `ls pubspec.yaml 2>/dev/null` | flutter-reviewer | 存在 Flutter 项目 |
| `ls manage.py 2>/dev/null` | django-reviewer | 存在 Django 项目入口 |
| `grep -r "fastapi" requirements.txt setup.py pyproject.toml 2>/dev/null \| head -1` | fastapi-reviewer | 依赖里有 FastAPI |

**无匹配时**：只跑 5 个基础 agent，不浪费 token。

**多语言混合项目**（如前端 React + 后端 Go）：同时追加 react-reviewer + go-reviewer。

## 可选 agent（按项目特征触发，不按语言）

`performance-optimizer` 默认不跑（455 行，较重）。以下情况**自动开启**：

- SPEC 里有性能指标（QPS/延迟/吞吐量）
- 代码里检测到数据库查询循环（grep `for.*\.query\|while.*SELECT`）
- 代码里有大数据处理（`readFile` + 循环、流式处理）
- 用户说"性能优化"或"这个功能要快"

开启后追加到诊断链末尾：

```
performance-optimizer → 性能瓶颈扫描（O(n²)/慢查询/内存泄漏/无效缓存）
```

## 问题类型说明

| type | 含义 | 处理方式 |
|---|---|---|
| `spec_violation` | 方案写了但代码没做到 | 必须修 |
| `quality_debt` | 跟方案一致但代码质量差 | 指派 agent 重构 |
| `error_handling` | 吞异常/空catch/无超时/部分写 | 指派 silent-failure-hunter 修 |
| `design_gap` | 方案没覆盖到的地方 | 补设计 |
| `perf_hotspot` | 性能瓶颈（仅当开启了第6 agent） | 指派 performance-optimizer |
| `dead_code` | 未使用的导入/变量/函数/文件 | 指派 refactor-cleaner |
| `normal` | 没问题 | 跳过 |

### 严重度定义

| 级别 | 含义 |
|---|---|
| P0 | 必须修，否则不上线（安全漏洞/功能缺失） |
| P1 | 建议修，影响可维护性（代码质量/测试覆盖） |
| P2 | 有余力再修（风格/注释等） |

## 批次数说明

清单不分批，但标注每个发现的 `complexity: easy/medium/hard` 字段，方便下游决定执行顺序。

## 角色分配说明

`assign` 字段的值直接对应 agent 名称。当前预置映射：

| assign 值 | 对应 agent | 用途 |
|---|---|---|
| security-reviewer | 安全审查 | SQL注入/密钥泄露/XSS |
| code-simplifier | 代码简化 | 长函数/重复代码/复杂逻辑 |
| silent-failure-hunter | 错误处理 | 吞异常/空catch/无日志 |
| tdd-guide | 测试驱动 | 补充测试 |
| architect | 架构审查 | 架构偏离 |
| code-reviewer | 代码审查 | 通用代码质量 |
| refactor-cleaner | 死代码清理 | 未使用的导入/变量/函数 |
| performance-optimizer | 性能优化 | 慢查询/大循环/内存泄漏 |
| python-reviewer | Python审查 | Python代码专项 |
| typescript-reviewer | TypeScript审查 | TS/TSX代码专项 |
| react-reviewer | React审查 | React组件专项 |
| go-reviewer | Go审查 | Go代码专项 |
| rust-reviewer | Rust审查 | Rust代码专项 |
| java-reviewer | Java审查 | Java/Spring代码专项 |
| cpp-reviewer | C++审查 | C++代码专项 |
| csharp-reviewer | C#审查 | C#代码专项 |
| swift-reviewer | Swift审查 | Swift代码专项 |
| kotlin-reviewer | Kotlin审查 | Kotlin/Android代码专项 |
| flutter-reviewer | Flutter/Dart审查 | Dart代码专项 |
| django-reviewer | Django审查 | Django/Python代码专项 |
| fastapi-reviewer | FastAPI审查 | FastAPI/Python代码专项 |

更多 agent 可根据 `~/.claude/agents/` 中的文件动态适配。

## 产物路径

```
<项目根目录>/.scratch/refactor-tasks/
├── tasks.yaml          ← 主任务清单（机器可读）
├── checklist.md        ← 人类可读的检查清单
└── raw-findings/       ← 各 agent 的原始输出（按实际运行顺序，仅供追溯）
    ├── 01-architect.md
    ├── 02-code-reviewer.md
    ├── 03-security-reviewer.md
    ├── 04-silent-failure-hunter.md
    ├── 05-pr-test-analyzer.md
    ├── 06-<语言agent>.md        ← 仅在检测到对应语言时存在（如 python-reviewer）
    └── 07-performance-optimizer.md  ← 仅在开启时存在
```

## 执行说明

每个 agent 调用使用 Agent tool，subagent_type 为上述 agent 名称。

prompt 中必须包含以下上下文，缺一不可：

```
=== 项目需求（PRD）===
【这里放 PRD.md 的关键内容】

=== 技术方案（SPEC）===  
【这里放 SPEC.md 的关键内容】

=== 审查范围 ===
【项目根目录的全部代码】

=== 审查任务 ===
【对照上述需求和方案，找出实现偏离、质量问题、安全隐患】
```

每个 agent 审查完后，提取其产出中的结构化发现（文件:行号:问题），写入对应的 raw-findings 文件。

全部 N 个 agent（5 基础 + 语言 agent + 可选性能 agent）完成后，合并去重，按 severity 排序，写入 tasks.yaml。

每次调用后打印**摘要卡**，标明本次诊断用了哪些 agent、发现了多少问题。

产出完成后打印：
```
✅ 诊断完成
  发现总数: N 条（P0: N, P1: N, P2: N）
  清单路径: .scratch/refactor-tasks/tasks.yaml
  建议下一步: 压缩上下文 → 开新会话 → /refactor-execute
```
