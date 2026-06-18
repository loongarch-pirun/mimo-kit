# Rules — 58 条编码规则

按领域分组，平台无关。

## Common（通用，10 条）

| 文件 | 覆盖 |
|------|------|
| agents.md | Agent 编排与使用指南 |
| code-review.md | 代码审查标准与检查清单 |
| coding-style.md | 编码风格（KISS/DRY/YAGNI/不可变性） |
| development-workflow.md | 开发工作流（plan → TDD → review） |
| git-workflow.md | Git 提交规范 |
| hooks.md | Hook 系统架构指南 |
| patterns.md | 设计模式（Repository / API 响应格式） |
| performance.md | 性能优化与模型选择 |
| security.md | 安全基线（OWASP / 密钥管理） |
| testing.md | 测试要求（80%+ 覆盖 / TDD） |

## TypeScript（5 条）

| 文件 | 覆盖 |
|------|------|
| coding-style.md | TypeScript 编码风格 |
| hooks.md | TypeScript 项目 Hook 配置 |
| patterns.md | TypeScript 设计模式 |
| security.md | TypeScript 安全（XSS/注入/类型安全） |
| testing.md | TypeScript 测试规范 |

## Web / 前端（7 条）

| 文件 | 覆盖 |
|------|------|
| coding-style.md | 前端编码风格（CSS/语义 HTML/文件组织） |
| design-quality.md | 设计质量标准（反模板策略） |
| hooks.md | 前端 Hook 配置 |
| patterns.md | 前端设计模式（Compound/Container/URL as State） |
| performance.md | 前端性能（Core Web Vitals / 打包预算） |
| security.md | 前端安全（CSP / XSS / 第三方脚本） |
| testing.md | 前端测试（视觉回归 / E2E / 可访问性） |

## 使用方式

```
在支持 rules 目录的平台上 → 直接放入对应目录
在不支持的平台上 → 把相关规则文件内容读入 system prompt
                   或按需在对话中引用
```
