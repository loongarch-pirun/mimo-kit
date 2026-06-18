# 系统级编码规范

做架构决策时应用。来源: Mythical Man-Month §1-5, SE at Google §1-4, Clean Architecture §5-7

---

## S01: 概念完整性

**级别: mandatory**

> 系统该让人觉得是一个人设计的。不是委员会投票出来的功能堆。
> — Brooks, Mythical Man-Month §4

**检查清单:**
- [ ] 新加的功能沿用项目已有的架构模式？
- [ ] 核心抽象由一个人 (或一个小团队) 设计？
- [ ] 没有 "三套不同的 error handling 混写" 的问题？
- [ ] 没有 "这部分像 Clean Architecture，那部分像 DDD，另一块像脚本" 的不一致？

**系统级反例:** 微服务体系 —— 两个服务做同一件事写得完全不一样，因为两个团队没统一样式。

---

## S02: 第二系统效应

**级别: mandatory**

> 第二个系统往往是最危险的系统。设计师把所有他在第一个系统中被迫省掉的东西全部加进去——结果过载。
> — Brooks, Mythical Man-Month §5

**检查清单:**
- [ ] 你是在做 v2 吗？如果是——你加进去的功能都是"用户现在就需要的"，还是 "上次没做成" 的遗憾？
- [ ] 设计里有没有 "做这个很酷" 的组件，但业务实际当前不需要？
- [ ] 当前版本的功能数跟上一版差超过 2 倍吗？

**反例:** 从 CLI 工具升级成 "带 Web UI、插件系统、云同步" 的平台 —— 结果一个都没做好。

---

## S03: 大教堂与集市

**级别: strongly-preferred | 源: Brooks, Mythical Man-Month §18 (No Silver Bullet)**

> Build one to throw away, you will anyhow.
> — Brooks

第一个版本是摸索问题本质的。丢了重写不是浪费——是不让摸索期的设计债绑住后续速度。

**检查清单:**
- [ ] 核心领域理解了（不是猜的）才开始建永久的架构？
- [ ] 原型被当成了原型丢掉，而不是 "加速上线后重构" (然后永远不重构)？

---

## S04: 海勒姆法则

**级别: mandatory | 源: Winters et al., SE at Google §1**

> With a sufficient number of users, every observable behavior of your system — including bugs, undocumented side effects, and error message text — will become depended upon by someone.

**检查清单:**
- [ ] 改了返回格式 (字段名、排序) → 全量调用方都确认了？
- [ ] 改了错误信息文本 → 有没有人在正则匹配这个文本做决策？
- [ ] 改了默认值 → 有没有调用方隐式依赖这个默认值？
- [ ] 改了未在文档里写的 "半公开" API → 你确定没人用吗？

**推论:** 公共 API 的每个 observable 行为都是接口。不文档化的接口会用 "事实上的文档" (代码使用方式) 来传播。

---

## S05: 无尽升级链

**级别: strongly-preferred | 源: Winters et al., SE at Google §21**

> Diamond dependency: A → B,C; B → D@v1; C → D@v2 → runtime conflict. The larger the codebase, the more the dependency graph forces upgrade coordination across unrelated teams.

**检查清单:**
- [ ] 第三方依赖的版本更新不需要三个团队开会协调？
- [ ] 核心依赖 (日志库/HTTP 客户端/序列化) 版本由平台统一管，不是每个服务各自定？
- [ ] 直接依赖传播 (不 pin 传递依赖的旧版本)？

---

## S07: 等待权力 (Power Awaits)

**级别: strongly-preferred | 源: Hunt & Thomas, Pragmatic Programmer §2**

> Don't decide now what you can defer. Every decision you postpone today costs nothing; every decision you make today costs a change later.
>
> "When in doubt, leave it out." — Hunt & Thomas

不做的成本是 0。做了又改回来的成本翻倍。

**检查清单:**
- [ ] 这个架构决策是不可逆的吗？不是 → 可以等到更了解再做。
- [ ] 目前有足够的上下文来做决策吗？不够 → 推迟。

---

## S08: 架构决策记录

**级别: mandatory | 源: Winters et al., SE at Google §7**

> Every non-trivial architectural decision must record: what was the context, what options were considered, why this option was chosen, what tradeoffs were accepted.

**格式:**
```markdown
# ADR-001: Title
- Status: proposed / accepted / superseded
- Context: 现状 + 问题
- Options considered: A, B, C
- Decision: B — 理由...
- Consequences: 得到了 X, 失去了 Y
```

**检查清单:**
- [ ] 架构决策放在 `docs/adr/` 或 `.claude/adr/` 下？
- [ ] 日期 + 编号 + 状态 + 选项记录？

---

## S09: 并行开发

**级别: strongly-preferred | 源: Brooks, Mythical Man-Month §2 + SE at Google §19**

> Adding people to a late project makes it later.
> — Brooks's Law

不是因为人不用干活——是新加的人需要沟通成本。每个新人必须与已有的每个人同步，团队通信带宽 O(n²)。

**检查清单:**
- [ ] 模块边界可以独立开发互不干扰？
- [ ] 能让两个开发者并行各写一个模块，各跑全量测试，不碰冲突？
- [ ] 系统是否可以在一个分支上开发、另一个分支上部署 —— 不互相踩脚？

---

## S10: 流动胜于完美

**级别: strongly-preferred | 源: Code Complete §5 + SE at Google §24**

> Design is not an upfront phase — it's a continuous activity that happens during construction.

**检查清单:**
- [ ] 架构决策是活的文档 (ADR, 代码注释, wiki) 而不是一次性的 PDF？
- [ ] 代码结构和架构图一致吗 (还是架构图早已过时)？
- [ ] 设计评审是不是做到代码写完了，还是只做在 "设计阶段" 一个时间段？

---

## 与规则的协同

`patterns.md` 定义了 Repository Pattern 和 API Response Format。这两条跟系统级规范完全兼容:
- Repository: 实现 DIP (M04) 的模式之一
- API Response Format: 统一 envelope → 概念完整性 (S01) 在 API 层的体现

`development-workflow.md` 的 "先计划再码" 跟 S08 (ADR) 互补:
- development-workflow: 管开发流程 (Plan → TDD → Review → Commit)
- S08: 管架构决策的文档化和追溯

`agents.md` 的 architect agent: 设计系统时用。本规范的 S01-S10 是 architect agent 做决策时的检查清单。
