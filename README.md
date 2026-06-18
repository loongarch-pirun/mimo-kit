<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License: MIT">
  <img src="https://img.shields.io/badge/agents-56-green" alt="56 Agents">
  <img src="https://img.shields.io/badge/workflows-19-orange" alt="19 Workflows">
  <img src="https://img.shields.io/badge/rules-58-red" alt="58 Rules">
  <img src="https://img.shields.io/badge/standards-53-purple" alt="53 Standards">
</p>

<h1 align="center">mimo-kit</h1>
<p align="center"><strong>跨平台 AI 知识包 · Cross-Platform AI Knowledge Kit</strong></p>
<p align="center">56 个 agent 定义 + 19 个工作流 + 58 条编码规则 + 53 条建造标准。<br>纯 Markdown，零平台绑定。</p>

---

## 简介 | Introduction

mimo-kit 是一个**平台无关的 AI 辅助开发知识包**。它将工程最佳实践编码为机器可读的 Markdown 文件——AI 代理定义、工作流管线、编码规则和工程标准——可以嵌入任何支持 system prompt 或规则目录的 AI 编码工具中。

**核心理念：** 纯文本，零绑定，按需加载，版本可控。

### 适用平台

| 平台 | 集成方式 |
|------|---------|
| **Claude Code** | `cp -r agents/ ~/.claude/agents/` |
| **Open Code / forks** | 30 行 loader 读取 agent .md 文件 |
| **WorkBuddy / Cursor** | 读入 SKILL.md / rules 目录 |
| **MiMo Code** | 读入对应模式指令 |
| **自定义 API 调用** | 按任务类型加载对应文件拼入 system prompt |

---

## 功能概览 | Feature Overview

### 🧠 56 个专项 AI Agent

| 类别 | 数量 | 覆盖 |
|------|------|------|
| 代码审查 | 9 | 通用、安全、静默故障、简化、重构、性能、注释、类型、测试 |
| 语言专项 | 16 | Python, TS, Go, Rust, C++, Java, C#, Swift, Kotlin, Flutter 等 |
| 构建修复 | 11 | C++/CMake, Go, Rust, Java, Kotlin, Swift, Dart, Django, React 等 |
| 架构设计 | 6 | 系统架构、功能架构、规划、代码探索、ML、网络 |
| 安全审计 | 5 | 全景测绘、卷积审计、压力测试、修复引擎、循环监控 |
| 文档知识 | 6 | 文档更新、库查询、SPEC 写作、PRD 写作、消息管理、对话分析 |
| 测试 | 2 | TDD 指南、E2E 运行器 |

### 🔧 19 个工作流管线

| 管线 | 子流程 |
|------|--------|
| **全流程管线** | orchestrate → brief → detail → draft-spec → decompose → plan-task → implement |
| **重构管线** | refactor-diagnose → refactor-execute |
| **安全审计管线** | cognitive-map → convolution-audit → repair-engine → stress-test |
| **自动触发** | coding-standard, caveman, diagnose, santa-method, skill-craft |

### 📐 58 条编码规则

- **通用 (10)** — Agent 编排、代码审查、编码风格、开发流程、Git 规范、设计模式、性能、安全、测试
- **TypeScript (5)** — 编码风格、Hooks、模式、安全、测试
- **Web/前端 (7)** — 编码风格、设计质量、Hooks、模式、性能、安全、测试

### 🏗️ 53 条建造标准

- **6 大建造原则** — 清晰性、内聚性、DRY、最小性、依赖方向、领域保真
- **命名规则** — 函数动词、类名词、领域语言、不缩写、同概念同名
- **模块规则** — 12 条模块级标准
- **函数规则** — 17 条函数级标准
- **错误处理** — 8 条错误处理标准
- **系统架构** — 9 条系统级架构标准

---

## 快速开始 | Quick Start

### 方案 A：作为本地知识包使用

```bash
# 克隆仓库
git clone https://github.com/YOUR_USER/mimo-kit.git
cd mimo-kit

# 在 Claude Code 中使用
cp -r agents/ ~/.claude/agents/
cp -r workflows/ ~/.claude/skills/
cp -r rules/ ~/.claude/rules/
```

### 方案 B：按需加载（自定义 API）

```python
def load_kit():
    kit = ""
    for f in ["agents/INDEX.md", "workflows/INDEX.md", "rules/INDEX.md"]:
        with open(f) as fp:
            kit += fp.read() + "\n"
    return kit

system_prompt = load_kit() + "\n请根据任务类型选择合适的 agent 或工作流。"
```

### 方案 C：按任务类型加载

```
1. 读 agents/INDEX.md → 了解有 56 个专项 agent
2. 读 workflows/INDEX.md → 了解有 19 个工作流
3. 需要执行某个工作流时 → 读对应 SKILL.md 做指令
4. 需要专项审查时 → 读对应 agent .md 做角色提示
5. 需要约束编码规范时 → 读 rules/ 或 standards/ 内容
```

---

## 项目结构 | Project Structure

```
mimo-kit/
├── KIT.md               # 入口文件
├── agents/              # 56 个 AI Agent 定义
│   ├── INDEX.md
│   ├── code-reviewer.md
│   ├── security-reviewer.md
│   ├── rust-reviewer.md
│   └── ...
├── workflows/           # 19 个工作流
│   ├── INDEX.md
│   ├── implement/SKILL.md
│   ├── refactor-diagnose/SKILL.md
│   └── ...
├── rules/               # 58 条编码规则
│   ├── INDEX.md
│   ├── common/
│   ├── typescript/
│   └── web/
├── standards/           # 53 条建造标准
│   ├── principles.md
│   ├── naming-rules.md
│   ├── function-rules.md
│   ├── module-rules.md
│   ├── error-rules.md
│   └── system-rules.md
├── LICENSE              # MIT License
├── README.md            # 本文件
├── CONTRIBUTING.md      # 贡献指南
├── CODE_OF_CONDUCT.md   # 行为准则
├── CHANGELOG.md         # 变更记录
└── .github/             # GitHub 模板
    ├── ISSUE_TEMPLATE.md
    └── PULL_REQUEST_TEMPLATE.md
```

---

## 设计原则 | Design Principles

- **纯文本，零绑定** — 所有文件是标准 Markdown，不依赖任何平台特性
- **自解释** — 每个目录有 INDEX.md 说明里面有什么、怎么用
- **按需加载** — 不需要全部读入上下文，按任务类型加载对应文件
- **版本可控** — 用 git 管理版本，`git tag` 标记版本号

---

## 路线图 | Roadmap

- [ ] 添加更多语言专项 Agent（Ruby, PHP, Scala, Zig）
- [ ] 提供 Docker 一键加载镜像
- [ ] CI/CD 集成示例（GitHub Actions）
- [ ] VS Code 扩展市场发布
- [ ] 社区 Agent 贡献模板

---

## 贡献 | Contributing

欢迎贡献！请阅读 [CONTRIBUTING.md](CONTRIBUTING.md) 了解详情。

## 许可证 | License

MIT — 随便用、随便改、随便传。详见 [LICENSE](LICENSE)。
