<h1 align="center">mimo-kit</h1>
<p align="center"><strong>Cross-Platform AI Knowledge Kit</strong></p>
<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="License: MIT">
  <img src="https://img.shields.io/badge/agents-56-green" alt="56 Agents">
  <img src="https://img.shields.io/badge/workflows-19-orange" alt="19 Workflows">
  <img src="https://img.shields.io/badge/rules-58-red" alt="58 Rules">
  <img src="https://img.shields.io/badge/standards-53-purple" alt="53 Standards">
</p>

<p align="center">
  <a href="README.md">中文</a> · <b>English</b>
</p>

<p align="center">56 agent definitions + 19 workflows + 58 coding rules + 53 engineering standards.<br>Pure Markdown, zero platform lock-in.</p>

---

## Introduction

mimo-kit is a **platform-agnostic AI-assisted development knowledge kit**. It encodes engineering best practices into machine-readable Markdown files — AI agent definitions, workflow pipelines, coding rules, and engineering standards — that can be embedded into any AI coding tool that supports system prompts or rule directories.

**Core philosophy:** Plain text, zero binding, on-demand loading, version-controllable.

### Supported Platforms

| Platform | Integration |
|----------|-------------|
| **Claude Code** | `cp -r agents/ ~/.claude/agents/` |
| **Open Code / forks** | 30-line loader reads agent .md files |
| **WorkBuddy / Cursor** | Load into SKILL.md / rules directory |
| **MiMo Code** | Load into mode instructions |
| **Custom API** | Load files by task type into system prompt |

## Feature Overview

### 🧠 56 Specialized AI Agents

| Category | Count | Coverage |
|----------|-------|----------|
| Code Review | 9 | General, security, silent-failure, simplification, refactoring, performance, comments, types, tests |
| Language-Specific | 16 | Python, TS, Go, Rust, C++, Java, C#, Swift, Kotlin, Flutter, etc. |
| Build Resolution | 11 | C++/CMake, Go, Rust, Java, Kotlin, Swift, Dart, Django, React, etc. |
| Architecture | 6 | System architecture, feature architecture, planning, code exploration, ML, networking |
| Security Audit | 5 | Cognitive map, convolution audit, stress test, repair engine, loop operator |
| Documentation | 6 | Doc updates, library lookup, SPEC writing, PRD writing, comms, conversation analysis |
| Testing | 2 | TDD guide, E2E runner |

### 🔧 19 Workflow Pipelines

| Pipeline | Sub-flows |
|----------|-----------|
| **Full Pipeline** | orchestrate → brief → detail → draft-spec → decompose → plan-task → implement |
| **Refactoring Pipeline** | refactor-diagnose → refactor-execute |
| **Security Audit Pipeline** | cognitive-map → convolution-audit → repair-engine → stress-test |
| **Auto-Triggered** | coding-standard, caveman, diagnose, santa-method, skill-craft |

### 📐 58 Coding Rules

- **Common (10)** — Agent orchestration, code review, coding style, dev workflow, Git conventions, design patterns, performance, security, testing
- **TypeScript (5)** — Coding style, hooks, patterns, security, testing
- **Web/Frontend (7)** — Coding style, design quality, hooks, patterns, performance, security, testing

### 🏗️ 53 Engineering Standards

- **6 Core Principles** — Clarity, cohesion, DRY, minimalism, dependency direction, domain fidelity
- **Naming Rules** — Functions as verbs, classes as nouns, domain language, no abbreviations, same concept same name
- **Module Rules** — 12 module-level standards
- **Function Rules** — 17 function-level standards
- **Error Handling** — 8 error-handling standards
- **System Architecture** — 9 system-level standards

## Quick Start

### Option A: Local Knowledge Kit

```bash
git clone https://github.com/loongarch-pirun/mimo-kit.git
cd mimo-kit

# Claude Code
cp -r agents/ ~/.claude/agents/
cp -r workflows/ ~/.claude/skills/
cp -r rules/ ~/.claude/rules/
```

### Option B: On-Demand Loading (Custom API)

```python
def load_kit():
    kit = ""
    for f in ["agents/INDEX.md", "workflows/INDEX.md", "rules/INDEX.md"]:
        with open(f) as fp:
            kit += fp.read() + "\n"
    return kit

system_prompt = load_kit() + "\nSelect appropriate agent or workflow based on task type."
```

### Option C: Load by Task Type

```
1. Read agents/INDEX.md → learn about 56 specialized agents
2. Read workflows/INDEX.md → learn about 19 workflows
3. To execute a workflow → read the corresponding SKILL.md
4. To review code → read the corresponding agent .md for role prompting
5. To enforce coding standards → read rules/ or standards/ content
```

## Project Structure

```
mimo-kit/
├── KIT.md               # Entry file
├── agents/              # 56 AI agent definitions
│   ├── INDEX.md
│   ├── code-reviewer.md
│   ├── security-reviewer.md
│   ├── rust-reviewer.md
│   └── ...
├── workflows/           # 19 workflows
│   ├── INDEX.md
│   ├── implement/SKILL.md
│   ├── refactor-diagnose/SKILL.md
│   └── ...
├── rules/               # 58 coding rules
│   ├── INDEX.md
│   ├── common/
│   ├── typescript/
│   └── web/
├── standards/           # 53 engineering standards
│   ├── principles.md
│   ├── naming-rules.md
│   ├── function-rules.md
│   ├── module-rules.md
│   ├── error-rules.md
│   └── system-rules.md
├── LICENSE              # MIT License
├── README.md            # 中文文档
├── README-EN.md         # English docs
├── CONTRIBUTING.md      # Contribution guide
├── CODE_OF_CONDUCT.md   # Code of conduct
└── CHANGELOG.md         # Changelog
```

## Ecosystem

mimo-kit works seamlessly with:

- [**hua-think-lite**](https://github.com/loongarch-pirun/hua-think-lite) — Pre-write validation + post-write security audit
- [**spec-guard**](https://github.com/loongarch-pirun/spec-guard) — SPEC-to-code alignment checking

## Design Principles

- **Plain text, zero binding** — All files are standard Markdown, no platform-specific features
- **Self-explanatory** — Each directory has INDEX.md explaining contents and usage
- **On-demand loading** — No need to load everything into context; load by task type
- **Version control** — Managed with git, versioned with `git tag`

## Roadmap

- [ ] More language-specific agents (Ruby, PHP, Scala, Zig)
- [ ] Docker one-click load image
- [ ] CI/CD integration examples (GitHub Actions)
- [ ] VS Code extension marketplace release
- [ ] Community agent contribution template

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
