# mimo-kit — 跨平台 AI 知识包

56 个 agent 定义 + 19 个工作流 + 58 条编码规则 + 53 条建造标准。纯 Markdown，零平台绑定。

## 内容

```
mimo-kit/
├── KIT.md              ← 入口文件
├── agents/
│   ├── INDEX.md        ← 56 agent 索引（按场景分类）
│   ├── code-reviewer.md
│   ├── security-reviewer.md
│   ├── rust-reviewer.md ... 共 56 个 .md
├── workflows/
│   ├── INDEX.md        ← 19 个工作流索引
│   ├── implement/SKILL.md
│   ├── refactor-diagnose/SKILL.md ... 共 19 个目录
├── rules/
│   ├── INDEX.md        ← 58 条规则索引
│   ├── common/         ← 10 条通用规则
│   ├── typescript/     ← 5 条 TS 专属规则
│   └── web/            ← 7 条前端规则
└── standards/
    ├── function-rules.md   ← 函数级 17 条
    ├── module-rules.md     ← 模块级 12 条
    ├── naming-rules.md     ← 命名 7 条
    ├── error-rules.md      ← 错误处理 8 条
    ├── system-rules.md     ← 系统架构 9 条
    ├── principles.md       ← 6 大建造原则
    └── source-accounting.md
```

## 在各平台上的使用

### Claude Code

```bash
cp -r agents/ ~/.claude/agents/
cp -r workflows/ ~/.claude/skills/
cp -r rules/ ~/.claude/rules/
```

直接生效。

### Open Code / 及 fork

写一个 30 行 loader，把 agents/ 目录下的 .md 文件按需读入 system prompt：

```python
import os
agents = {}
for f in os.listdir("agents"):
    if f.endswith(".md") and f != "INDEX.md":
        name = f.replace(".md", "")
        with open(f"agents/{f}") as fp:
            agents[name] = fp.read()
# 需要审查 Python 代码时：
system_prompt += agents["python-reviewer"]
```

### MiMo Code

把 agent .md 或 workflow SKILL.md 的内容作为指令读入对应模式。

### 通用方式（任何平台）

```
1. 读 agents/INDEX.md → 让模型知道你有 56 个专项 agent
2. 读 workflows/INDEX.md → 让模型知道你有 19 个工作流
3. 需要执行某个工作流时 → 读对应 SKILL.md 内容做指令
4. 需要专项审查时 → 读对应 agent .md 内容做角色提示
5. 需要约束编码规范时 → 读 rules/ 或 standards/ 内容
```

### 自己调 API

```python
def load_kit():
    kit = ""
    for f in ["agents/INDEX.md", "workflows/INDEX.md", "rules/INDEX.md"]:
        with open(f) as fp:
            kit += fp.read() + "\n"
    return kit

system_prompt = load_kit() + "\n请根据任务类型选择合适的 agent 或工作流。"
```

## 设计原则

- **纯文本，零绑定**：所有文件是标准 Markdown，不依赖任何平台特性
- **自解释**：每个目录有 INDEX.md 说明里面有什么、怎么用
- **按需加载**：不需要全部读入上下文，按任务类型加载对应文件
- **版本可控**：用 git 管理版本，`git tag` 标记版本号

## License

MIT — 随便用、随便改、随便传。
