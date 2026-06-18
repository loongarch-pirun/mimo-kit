# 源头溯源

每条规则可回溯到具体的书和章节。审查引用模式: `Book §Chapter: Principle`

---

## 书→规则映射

| 书 | 规则 ID | 贡献度 |
|---|---------|--------|
| **Code Complete** (McConnell) | F01-F10, F13-F23, M01, N01-N10, E01-E10 | 40% |
| **Clean Code / Clean Architecture** (Martin) | F01, F05, F06, F18, M01-M04, M07-M08, N04 | 25% |
| **The Pragmatic Programmer** (Hunt & Thomas) | F10, F11, F12, F14, F17, E03, E04, E07 | 15% |
| **Domain-Driven Design** (Evans) | M09-M11, N06 | 10% |
| **A Philosophy of Software Design** (Ousterhout) | M05-M06 | 5% |
| **Refactoring** (Fowler) | F01, F03, F14, F21, M03, M13 | 5% |

---

## 与规则的冲突解决

| 项目 | 规则 | 12 本经典 | 本 Skill 的裁决 |
|------|---------|----------|---------------|
| **不可变性** | 强制 (coding-style.md §1) | 推荐 (Pragmatic §2) | **规则优先** — 不可变性是硬约束，12 本经典的推荐级别不覆盖 |
| **函数大小上限** | 50 行 (代码审查标准) | 20 行 (Code Complete §7) | **经典优先** — F02 设为 20 行强制，超 20 行触发审查 |
| **嵌套上限** | 4 层 (coding-style) | 3 层 (Code Complete §19) | **经典优先** — F08 设为 3 层 |
| **文件大小** | 800 行 (coding-style) | 无明确限制 | **规则优先** — 文件超过 800 行由 Hook 硬阻断 |

---

## 没有被覆盖的工程经典

以下书在前两层 (函数/模块) 贡献有限，但在 **Layer 3 系统级** 得到实现:

| 书 | 系统级规则 | 贡献 |
|---|-----------|------|
| Mythical Man-Month (Brooks) | S01-S03, S06, S09 | 概念完整性、第二系统效应、盖尔定律、布鲁克斯定律 |
| SE at Google (Winters) | S04, S05, S08 | 海勒姆法则、无尽升级链、架构决策记录 |

以下书仍未被覆盖——它们是团队/组织级实践，非个人编码规范:
- Working Effectively with Legacy Code (Feathers): 遗留代码测试策略
- Art of Unit Testing (Osherove): 单元测试技巧
- xUnit Test Patterns (Meszaros): 测试模式参考
- How Google Tests Software: 组织级测试文化
