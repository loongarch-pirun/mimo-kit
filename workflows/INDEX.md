# Workflows — 19 个工作流

## 全流程管线（斜杠触发）

| 工作流 | 阶段 | 说明 |
|--------|------|------|
| orchestrate | 管线调度 | 7 步连续管线编排入口 |
| brief | 需求锚定 | 用户需求 → 原子化需求清单 |
| detail | 需求详化 | 逐条展开颗粒度 |
| draft-spec | 方案设计 | 产出 PRD + SPEC 文档 |
| decompose | 任务拆解 | 拆成 phases.yaml 依赖图 |
| plan-task | 执行规划 | 为当前 phase 每任务生成计划 |
| implement | 编码实现 | 逐 phase 写代码 + 门禁 |

## 重构管线（斜杠触发）

| 工作流 | 说明 |
|--------|------|
| refactor-diagnose | 检测项目语言 → 串行诊断 → 产出 tasks.yaml |
| refactor-execute | 读 tasks.yaml → 逐条调度 agent 修 → 验证 → checkpoint |

## 安全审计管线（斜杠触发）

| 工作流 | 说明 |
|--------|------|
| cognitive-map | 全景代码库测绘（6+2 维扫描） |
| convolution-audit | 卷积安全审计（trivy + gitleaks 驱动） |
| repair-engine | 修复编排（接收问题 → 修复 → 收敛） |
| convolution-stress-test | 四阶段极限压测 |

## 测试验证（斜杠触发）

| 工作流 | 说明 |
|--------|------|
| go-board | VM 混沌测试（ChaosBlade + k6 攻击验证） |

## 自动触发

| 工作流 | 触发条件 | 说明 |
|--------|---------|------|
| coding-standard | 写代码 / 重构 / 新建类 | 53 条编码规范自动应用 |
| caveman | caveman mode / less tokens | 超精简通信模式 |
| diagnose | diagnose this / debug this | 结构化 Bug 诊断循环 |
| santa-method | 双盲审查 / santa check | 对抗式双模型审查 |
| skill-craft | check skill / 评估 skill | Skill 质量评估与修复 |

## 使用方式

```
在支持 slash command 的平台上 → 直接打 /implement 等
在不支持的平台上 → 读取对应 SKILL.md 文件内容
                   按流程描述手动执行各步骤
```
