# 模块级编码规范

设计类和模块时应用。来源: Clean Architecture §3-6, DDD §4-7, Philosophy of SD §4-6, Code Complete §5-6

---

## M01: 单一职责 (类级)

**级别: mandatory**

> 一个类只有一个理由去改它。
> — Martin, Clean Architecture SRP

**检查清单:**
- [ ] 这个类的每个方法都跟类名描述的职责直接相关？
- [ ] 改一个业务流程不需要动这个类的多个不相关方法？
- [ ] 类名描述里没有 "and"？

---

## M02: 开闭原则

**级别: strongly-preferred**

> 对扩展开放，对修改封闭。加行为不碰已有代码。
> — Martin, Clean Architecture OCP

**检查清单:**
- [ ] 加一个新变种 (新支付方式/新导出格式) 是加新类，不是改已有类？
- [ ] 用策略模式/插件/多态而不是 if-else 链区分变种？

---

## M03: 接口隔离

**级别: strongly-preferred**

> 不强迫调用方依赖它不需要的方法。胖接口该拆。
> — Martin, Clean Architecture ISP

**检查清单:**
- [ ] 没有调用方只用一个接口的 2/5 方法？
- [ ] 没有接口方法在某个实现里抛 UnsupportedOperationException？

---

## M04: 依赖反转

**级别: mandatory**

> 高层不依赖低层。两者都依赖抽象。抽象不依赖细节，细节依赖抽象。
> — Martin, Clean Architecture DIP

**检查清单:**
- [ ] 领域代码 (entity/usecase) 不 import 基础设施 (database/http/queue)？
- [ ] 基础设施实现领域定义的接口，而非反过来？
- [ ] 模块间的边界是接口，不是具体类？

---

## M05: 深模块

**级别: strongly-preferred**

> 模块越深越好: 接口从外面看极简，实现可以复杂。接口复杂但实现简单的是浅模块——封装没创造价值。
> — Ousterhout, Philosophy of SD §4

**检查清单:**
- [ ] 模块的公开接口 ≤3 个方法？
- [ ] 公开方法签名简单 (参数少、不暴露内部类型)？
- [ ] "用这个模块" 不需要先读源代码？

---

## M06: 信息隐藏

**级别: mandatory**

> 模块内部的设计决策不该泄露到接口。改了文件格式/数据库结构/缓存策略，调方不该知道。
> — Ousterhout, Philosophy of SD §5

**检查清单:**
- [ ] 没有 "把数据库连接字符串传给领域服务" 的写法？
- [ ] 没有 "用这个常量前你得先知道实现细节" 的写法？
- [ ] 换了存储引擎，调方代码不需要改？

---

## M07: 非循环依赖

**级别: mandatory**

> A 依赖 B，B 绝对不依赖 A。不管多远，循环就是禁区。
> — Martin, Clean Architecture ADP

**检查清单:**
- [ ] 没有循环 import？
- [ ] 依赖图可以画出来，箭头全指向同一方向？

---

## M08: 稳定依赖稳定度

**级别: strongly-preferred**

> 越稳定的模块依赖越不稳定的模块就是反方向。不稳定 (常变的) 应该在依赖图外层。
> — Martin, Clean Architecture SDP

**检查清单:**
- [ ] 广泛使用的工具类很少改？
- [ ] 频繁变更的模块不被核心领域依赖？
- [ ] 废弃率高的第三方库被封装在一处，不在代码库中到处引用？

---

## M09: 聚合根

**级别: mandatory (领域模块)**

> 聚合内的对象整体读、整体写。聚合外只能通过根引用内部——不穿透。
> — Evans, DDD §6

**检查清单:**
- [ ] 外面不能直接修改聚合内部的子对象 (OrderLine 只能 via Order)？
- [ ] 一个事务只动一个聚合？
- [ ] 聚合之间用 ID 引用，不直接持有对象引用？

---

## M10: 限界上下文

**级别: mandatory (多子系统)**

> 不同上下文的模型不同。Sales 的 Customer 不同于 Support 的 Customer——不共享同一类。
> — Evans, DDD §14

**检查清单:**
- [ ] 两个团队/子域用不同的术语 → 代码模型也不同？
- [ ] 跨上下文通信走 ACL (Anti-Corruption Layer) 而不是直接引同一个类？
- [ ] 没有 "一个 User 类跨 5 个模块各加字段" 的写法？

---

## M11: 值对象

**级别: strongly-preferred**

> 纯粹由值定义的、不可变的、可替换的概念 (Money, Email, Address) 做成值对象，不给 ID 和生命周期。
> — Evans, DDD §5

**检查清单:**
- [ ] Money/Email/Address/Quantity 是无 ID 的不可变值对象？
- [ ] 值对象之间可比较 (equals/hashCode)？
- [ ] 没有将值对象写成 Entity (给 ID、有生命周期)？

---

## M12: 概念完整性

**级别: strongly-preferred**

> 系统该让人觉得是一个人设计的。同一模式用到底，不混用多种风格。
> — Brooks, Mythical Man-Month §4

**检查清单:**
- [ ] 新增的模块沿用项目已有的架构模式？
- [ ] 没有 "error handling 两种写法混用"？
- [ ] 没有 "Controller-Service-Repository 和 CQRS 各写一半"？


