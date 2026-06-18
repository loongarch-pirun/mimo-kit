# Contributing to mimo-kit

感谢你考虑为 mimo-kit 做出贡献！本项目的核心是社区驱动的工程最佳实践集合。

## 贡献方式

### 🐛 报告 Bug | Report Issues

- 使用 GitHub Issues 提交
- 清晰描述问题：发生了什么、期望的行为、复现步骤
- 附上相关文件路径和示例内容

### 💡 提交新 Agent / 工作流 / 规则

Agent、工作流、编码规则和建造标准都接受社区贡献：

1. **新 Agent** → 在 `agents/` 下创建 `<name>.md`，参考已有文件的前言格式
2. **新工作流** → 在 `workflows/` 下创建 `<name>/SKILL.md`
3. **新规则** → 在 `rules/` 对应类别下创建 `.md` 文件
4. **新标准** → 在 `standards/` 下创建 `.md` 文件

### 🔧 改进现有内容

- 修复拼写、语法或格式问题
- 补充缺失的引用或出处
- 优化提示词的清晰度和有效性

## 提交规范

### 分支命名

```
feat/add-rust-agent
fix/typo-in-principles
docs/improve-readme
```

### 提交消息格式

```
<type>: <简短描述>

<可选：详细说明>
```

**type 类型：**

| type | 说明 |
|------|------|
| `feat` | 新增 Agent / 工作流 / 规则 |
| `fix` | 修复错误或不准确的内容 |
| `docs` | 文档改进 |
| `style` | 格式调整（不影响含义） |
| `refactor` | 结构调整 |
| `chore` | 构建/工具链相关 |

### PR 流程

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feat/my-agent`)
3. 提交变更 (`git commit -m "feat: add Ruby agent"`)
4. 推送到你的分支 (`git push origin feat/my-agent`)
5. 创建 Pull Request

## 质量标准

提交的 Agent 或规则应满足：

- ✅ 格式与现有文件一致（YAML frontmatter + Markdown 正文）
- ✅ 内容自包含，不依赖未说明的外部工具
- ✅ 引用来源标注清晰（书籍、章节、工具版本）
- ✅ 中英文皆可，但保持一致性

## 行为准则

本项目的所有参与者应遵守 [CODE_OF_CONDUCT.md](CODE_OF_CONDUCT.md)。

---

再次感谢你的贡献！🎉
