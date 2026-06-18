# 六大建造原则

从 12 本工程经典提炼。每条原则对应一项代码腐烂风险的反面——审查找病，建造防病。

---

## P1: 清晰性 — 代码解释自己

**对应腐烂: R1 认知过载**

> "Code should be written to be read by humans first, and executed by machines second."
> — Steve McConnell, Code Complete §33

写代码为读代码的人。看到一段代码时应该立刻理解它做什么，不需要翻实现、不需要猜命名。

**源头:**

| 原则 | 书 | 章节 |
|------|-----|------|
| 函数短小、抽象层级一致 | McConnell, Code Complete | §7: High-Quality Routines |
| 命名说清意图 | McConnell, Code Complete | §11: Variable Names |
| 不写魔数 | McConnell, Code Complete | §12: Fundamental Data Types |
| 浅模块之反: 深模块 | Ousterhout, Philosophy of SD | §4: Modules Should Be Deep |
| 领域语言 | Evans, DDD | Ubiquitous Language |

## P2: 内聚性 — 一起变的放一起

**对应腐烂: R2 变更传播**

> "A class should have only one reason to change."
> — Robert C. Martin, Clean Architecture

改一个业务功能只动一处代码。因为散开来改，就有一处忘改，于是 bug 诞生。

**源头:**

| 原则 | 书 | 章节 |
|------|-----|------|
| 单一职责 | Martin, Clean Architecture | SRP |
| 霰弹式修改之反 | Fowler, Refactoring | Shotgun Surgery |
| 发散式变更之反 | Fowler, Refactoring | Divergent Change |
| 正交性 | Hunt & Thomas, Pragmatic Programmer | §2: Orthogonality |
| 信息隐藏 | Ousterhout, Philosophy of SD | §5: Information Hiding |

## P3: DRY — 每项知识只有单一表达

**对应腐烂: R3 知识重复**

> "Every piece of knowledge must have a single, unambiguous, authoritative representation within a system."
> — Hunt & Thomas, The Pragmatic Programmer

DRY 不是不重复代码行——是不重复决策。两段代码长得一样但理由不同，不算重复。

**源头:**

| 原则 | 书 | 章节 |
|------|-----|------|
| DRY | Hunt & Thomas, Pragmatic Programmer | DRY Principle |
| 重复代码 | Fowler, Refactoring | Duplicate Code |
| 统一语言 | Evans, DDD | Ubiquitous Language |
| 并行继承之反 | Fowler, Refactoring | Parallel Inheritance |

## P4: 最小性 — 不写一行不需要的代码

**对应腐烂: R4 意外复杂度**

> "You aren't gonna need it."
> — Kent Beck (via Fowler, Refactoring)

不建"将来说不定会用"的抽象。不为了对称多写层。代码比问题复杂的时候就该删了——不是加注释解释。

**源头:**

| 原则 | 书 | 章节 |
|------|-----|------|
| YAGNI | McConnell, Code Complete | §5: Design in Construction |
| 投机普遍性之反 | Fowler, Refactoring | Speculative Generality |
| 懒类之反 | Fowler, Refactoring | Lazy Class |
| 第二系统效应 | Brooks, Mythical Man-Month | §5: The Second-System Effect |
| 战术编程之反 | Ousterhout, Philosophy of SD | §3: Strategic vs. Tactical |

## P5: 依赖方向 — 稳定向内，不稳定向外

**对应腐烂: R5 依赖紊乱**

> "Depend in the direction of stability."
> — Robert C. Martin, Clean Architecture

领域不依赖基础设施。高层不依赖低层。方向一致，循环是禁区。

**源头:**

| 原则 | 书 | 章节 |
|------|-----|------|
| 依赖反转 | Martin, Clean Architecture | DIP |
| 非循环依赖 | Martin, Clean Architecture | ADP |
| 稳定依赖 | Martin, Clean Architecture | SDP |
| 接口隔离 | Martin, Clean Architecture | ISP |
| 概念完整性 | Brooks, Mythical Man-Month | §4: Conceptual Integrity |

## P6: 领域保真 — 代码说业务的语言

**对应腐烂: R6 领域模型失真**

> "The model is the backbone of a language used by all team members."
> — Eric Evans, Domain-Driven Design

代码里的名字跟业务说的一模一样。领域对象有行为，不只是装数据的袋子。不是数据库表结构倒影到代码。

**源头:**

| 原则 | 书 | 章节 |
|------|-----|------|
| 统一语言 | Evans, DDD | Ubiquitous Language |
| 领域模型 | Evans, DDD | Domain Model Pattern |
| 贫血模型之反 | Evans, DDD | (反模式: Anemic Domain Model) |
| 数据类之反 | Fowler, Refactoring | Data Class |
| LSP | Martin, Clean Architecture | Liskov Substitution |

---

## 与规则的继承关系

这 6 条建造原则跟 58 个规则文件是**互补层**，不是替代:

```
规则层:   "不坏" — ❌ 不超 800 行, ❌ 不硬编码 secret, ❌ 不吞异常
建造原则层:   "好"   — ✅ 单函数只做一件事, ✅ 命名说清意图, ✅ 依赖向内
```

唯一的显式冲突——不可变性——已继承强制标准: `coding-style.md §1` 优先于 12 本经典的推荐级别。见 `source-accounting.md`。
