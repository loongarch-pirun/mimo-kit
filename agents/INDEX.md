# Agents — 56 个专项 Agent

按类别索引，方便快速定位。

## 代码审查

| Agent | 适用场景 |
|-------|---------|
| code-reviewer | 通用代码质量审查（加固版，6 维检查） |
| security-reviewer | 安全漏洞扫描（OWASP Top 10） |
| silent-failure-hunter | 错误处理缺陷扫描（空catch/吞异常/无超时） |
| code-simplifier | 代码简化（长函数/重复逻辑/复杂条件） |
| refactor-cleaner | 死代码清理（无用导入/变量/函数/文件） |
| performance-optimizer | 性能瓶颈（O(n²)/慢查询/内存泄漏） |
| comment-analyzer | 代码注释质量分析 |
| type-design-analyzer | 类型设计合理性分析 |
| pr-test-analyzer | PR 测试覆盖分析 |

## 语言专项审查

| Agent | 语言/框架 |
|-------|----------|
| python-reviewer | Python |
| typescript-reviewer | TypeScript / JavaScript |
| react-reviewer | React / JSX |
| go-reviewer | Go |
| rust-reviewer | Rust |
| cpp-reviewer | C++ |
| java-reviewer | Java / Spring Boot |
| csharp-reviewer | C# / .NET |
| swift-reviewer | Swift / iOS |
| kotlin-reviewer | Kotlin / Android |
| flutter-reviewer | Flutter / Dart |
| django-reviewer | Django / Python |
| fastapi-reviewer | FastAPI / Python |
| database-reviewer | PostgreSQL / Supabase / SQL |
| fsharp-reviewer | F# |
| a11y-architect | 无障碍 / WCAG 2.2 |

## 构建修复

| Agent | 语言/工具 |
|-------|----------|
| build-error-resolver | 通用构建错误 |
| cpp-build-resolver | C++ / CMake |
| go-build-resolver | Go |
| rust-build-resolver | Rust / Cargo |
| java-build-resolver | Java / Maven / Gradle |
| kotlin-build-resolver | Kotlin / Gradle |
| swift-build-resolver | Swift / Xcode |
| dart-build-resolver | Dart / Flutter |
| django-build-resolver | Django / Python |
| pytorch-build-resolver | PyTorch / CUDA |
| react-build-resolver | React / Vite / Webpack / Next.js |

## 架构与设计

| Agent | 用途 |
|-------|------|
| architect | 系统架构设计与审查 |
| code-architect | 功能架构设计（从代码库提取模式） |
| planner | 复杂功能实现规划 |
| code-explorer | 代码库深度探索与执行路径追踪 |
| mle-reviewer | 机器学习管线评审 |
| network-config-reviewer | 网络配置审查 |

## 安全审计与压测

| Agent | 用途 |
|-------|------|
| cognitive-map | 全景代码库测绘（6+2 维扫描） |
| convolution-audit | 卷积安全审计（trivy + gitleaks 驱动） |
| convolution-stress-test | 四阶段极限压力测试 |
| repair-engine | 修复编排（接收问题→修复→收敛） |
| loop-operator | 自主循环监控与干预 |

## 文档与知识

| Agent | 用途 |
|-------|------|
| doc-updater | 文档与 codemap 更新 |
| docs-lookup | 第三方库文档查询（Context7） |
| spec-writer | SPEC 技术方案写作 |
| prd-writer | PRD 需求文档写作 |
| chief-of-staff | 消息分类与通信管理 |
| conversation-analyzer | 对话分析（hook 行为提取） |

## 测试

| Agent | 用途 |
|-------|------|
| tdd-guide | 测试驱动开发 |
| e2e-runner | 端到端测试（Playwright） |

## 工具

| Agent | 用途 |
|-------|------|
| harness-optimizer | Harness 配置优化 |
| doc-updater | 文档与 codemap 同步 |

## 使用方式

```
场景：审查一段 Python 代码
→ 读取 agents/python-reviewer.md 内容
→ 拼入 system prompt
→ 让模型以该角色审查

场景：诊断项目问题
→ 按 refactor-diagnose 工作流执行
→ 自动检测项目语言 → 加载对应 agent
→ 串行诊断 → 产出任务清单
```
