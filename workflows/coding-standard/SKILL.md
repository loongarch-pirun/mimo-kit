---
name: coding-standard
description: >
  12 本工程经典驱动的编码规范 — 写代码时自动应用。基于 Code Complete、Clean Architecture、
  The Pragmatic Programmer、DDD、A Philosophy of Software Design 等经典著作。
  不是审查代码，是建造规范：写每个函数、每个模块、每个类的命名时自动检查。
  TRIGGER: 写代码 / 写新函数 / 新建类 / 重构 / 改命名 / 写模块 / 设计接口 / 写一个...
  DO NOT TRIGGER on: 代码审查 / 改 bug / 查问题 / 解释代码
---

# Coding Standard — 12 本工程经典的编码规范

写代码的时候自动应用。审查找病——这个防病。

## 快速参考

| 你写什么 | 参考什么 |
|----------|---------|
| 一个函数 | `function-rules.md` — 17 条规则 |
| 一个类/模块 | `module-rules.md` — 13 条规则 |
| 命名 | `naming-rules.md` — 7 条规则 |
| 错误处理 | `error-rules.md` — 8 条规则 |
| 架构决策 | `system-rules.md` — 9 条规则 |
| 设计方向 | `principles.md` — 六大建造原则 |
| 想查某条规则的出处 | `source-accounting.md` — 书→规则映射 |

示例对照在 `examples/` 下。

## 执行

写代码时:

1. **读 `_shared/function-rules.md`** 中与你正在写的代码类型相关的部分
2. **逐条对照** — 写的代码是否符合每条 mandatory 规则？
3. **违反 mandatory → 不写那行代码，改写法**
4. **违反 strongly-preferred → 可以写，但要能在注释里说出理由**

## 项目配置

如果项目根目录有 `.coding-standard.yaml`，按以下顺序应用:

1. `ignore` — 跳过的文件 glob
2. `disable` — 跳过的规则 ID
3. `severity` — 覆盖规则严重度
4. `focus` — 如果非空，只检查列出的规则
5. `suppress` — 已知例外项
6. `custom_rules` — 项目独有规则

如果 `.coding-standard.yaml` 不存在，全规则默认严重度运行。

## 与规则的协作

```
规则层:   "不坏" — 流程纪律、安全底线 (58 个规则文件)
本 Skill 层:  "好"   — 代码技艺、建造标准 (基于 12 本工程经典)

协作方式:
  - 写代码时: 本 Skill 先约束 (怎么写才算好)
  - 提交前: Hooks 再检查 (format/lint/type-check)
  - 发布前: Go-Board VM 攻击验证
```

## 严重度定义

| 级别 | 含义 | 违反后的行为 |
|------|------|------------|
| **mandatory** | 不允许违反 | 不写那行代码，换个写法 |
| **strongly-preferred** | 最少解释原因 | 可以写，但在注释里说理由 |
