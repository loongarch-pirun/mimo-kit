# 函数级编码规范

写每个函数时应用以下 17 条规则。按优先级排列：`mandatory` 的不允许违反，`strongly-preferred` 的最少解释原因。

来源: Code Complete §7-12, Clean Code §2-3, The Pragmatic Programmer §2-5, Refactoring §3

---

## F01: 单一职责

**级别: mandatory**

> 一个函数做一件事。另一件事就该是另一个函数。
> — Code Complete §7.1, Fowler Long Method

**检查清单:**
- [ ] 能用一句话描述函数职责，且句子里没有 "和" / "以及"？
- [ ] 函数名准确说了它做什么？
- [ ] 改一个业务规则不需要碰这个函数的好几个部分？

**反例:**
```python
def update_profile(user_id, name, email, avatar):
    user = db.query(f"SELECT * FROM users WHERE id={user_id}")
    user['email'] = email
    if user['email'] != email:  # bug: 上一步已覆写了
        smtp.send(email, "Email changed")
    points = user['login_count'] * 10 + 500
    db.execute(f"UPDATE loyalty SET points={points} WHERE user_id={user_id}")
```
这个函数做了四件事: 查用户、改邮箱、发通知、算积分。拆成四个。

---

## F03: 参数个数

**级别: mandatory**

> 不超过 3 个参数。超出就该考虑"这些参数是不是该成为一个对象"。
> — Code Complete §7.5

**检查清单:**
- [ ] 参数 ≤3？
- [ ] 超过 3 个，是否至少 2 个是 flag/option 且可合并成 config 对象？

**反例:** `createOrder(user, product, quantity, price, currency, shipTo, billingTo, notes)`  
→ 变成 `createOrder(user, productItems, orderOptions)`

---

## F05: 命名不撒谎

**级别: mandatory**

> 函数名承诺做什么，实现就得做什么。多做了就是撒谎。
> — Code Complete §11.5

**检查清单:**
- [ ] `getX()` 是纯获取，不会顺便改状态？
- [ ] `isValid()` 是纯判定，不会顺便写日志/发事件？
- [ ] `calculate()` 真的只算，不发请求/不存数据库？
- [ ] 名字里没提到的副作用，它真的没有？

---

## F06: 抽象层级一致

**级别: mandatory**

> 函数内部的每一行该在同一个抽象层级上。不要"查用户 → SQL → 发邮件"混在一个函数。
> — Code Complete §7.2, McConnell

**检查清单:**
- [ ] 函数内没有在一行做 HTTP 请求，下一行拼接 SQL 字符串？
- [ ] 没有"业务规则"混在"文件读写"中间？

---

## F07: 不用否定条件

**级别: mandatory**

> 大脑处理否定比肯定慢。`if is_valid` 比 `if !is_invalid` 快。
> — Code Complete §15.2

**检查清单:**
- [ ] 条件表达式不含 `!`, `not`, `unless`？
- [ ] 布尔变量名本身是正面的？`isValid` 而不是 `isNotInvalid`

---

## F10: 返回明确、不返回 null

**级别: mandatory**

> 不返回 null。返回 Optional/Maybe/空集合。null 是 NullPointerException 的邀请函。
> — Code Complete §8.3, Pragmatic Programmer §4

**检查清单:**
- [ ] 不返回 `null`, `nil`, `None`？
- [ ] 没找到时返 Optional.empty() / [] / Result type？
- [ ] 不是返回负值魔法数字 (-1, 999) 来表达错误？

---

## F11: CQS — 命令和查询分离

**级别: mandatory**

> 一个方法或是命令 (改状态，不返值)，或是查询 (返值，不改状态)。不同时做两件事。
> — Bertrand Meyer (by Fowler, Refactoring; Pragmatic Programmer §4)

**检查清单:**
- [ ] 返值的函数不改任何 observable state？
- [ ] 改 state 的函数返回 void / None / CommandResult？
- [ ] 没有 "getOrCreate" 这类名字？——那是命令，不是查询。

---

## F15: 纯函数优先

**级别: strongly-preferred**

> 能不依赖外部状态就不依赖。同样的输入永远出同样的输出——测试零成本。
> — Code Complete §7.3, Practical functional design

**检查清单:**
- [ ] 函数结果只依赖输入参数，不依赖全局变量/环境变量/文件系统？
- [ ] 有没有把 I/O (杂质) 推到函数最外层、核心逻辑提成纯函数？

---

## F16: 描述性变量

**级别: strongly-preferred**

> 复杂表达式拆成有名字的中间变量。`if user.age >= 18 and user.age <= 65` 优于 `if checkAge(user)` 吗？不——但 `is_eligible_for_insurance = 18 <= user.age <= 65` 比两者都好。
> — Code Complete §11.3

**检查清单:**
- [ ] 多条件 Boolean 表达式给了有名字的变量？
- [ ] 一行超过 3 个操作符的表达式拆成子变量？

---

## F17: 不写死路径

**级别: mandatory**

> 绝对不要拼接字符串做文件路径或 URL——参数化。
> — Code Complete §8.2, Pragmatic Programmer §4

**检查清单:**
- [ ] 文件路径不用字符串拼接？`base_dir / sub_dir / filename` 而非 `base + "/" + sub + "/" + name`
- [ ] URL 中无直接拼接用户输入？参数化或用 url-join 库。

---

## F18: 注释说"为什么"不是说"什么"

**级别: mandatory**

> 注释不该翻译代码。注释该解释写这段代码的理由、被舍弃的替代方案、非直观的性能考虑。
> — Code Complete §32.1, Clean Code §4

**检查清单:**
- [ ] 没有 "// increment i" 这类的注释？代码已经说了
- [ ] 对非直观逻辑有解释理由的注释？
- [ ] 被注释掉的死代码已删除？

---

## F19: 资源获取即释放

**级别: mandatory**

> 打开的文件/连接/锁在同一个函数里关。不是"调者负责关"——就是此处关。
> — Code Complete §8.3, Pragmatic Programmer §5

**检查清单:**
- [ ] `open()` / `connect()` / `lock()` 后面有对应的 `close()` / `release()` / `unlock()`？
- [ ] 清理逻辑在 finally/defer/context-manager 里，不受异常影响？

---

## F20: 无副作用注释

**级别: mandatory**

> 函数改了不属于它的东西，就得在名字或文档上说清楚。
> — Code Complete §7.3

**检查清单:**
- [ ] 改全局变量/静态字段的函数，名字里有体现？(如 `resetGlobalCache`)
- [ ] 名字看起来是纯读取但实质有写的，不存在？

---

## F21: 不暴露内部表示

**级别: mandatory**

> 函数不返回内部数据结构的引用——返回副本或不可变视图。
> — Code Complete §6.2

**检查清单:**
- [ ] 返回 `List` / `Map` 的函数，返的是不可变副本还是活引用？
- [ ] 没有人能通过返回值改到对象的内部状态？

---

## F22: 参数顺序一致

**级别: strongly-preferred**

> 相类函数的参数顺序相同。`copy(src, dst)` 和 `move(src, dst)` 不要变成 `move(dst, src)`。
> — Code Complete §7.5

**检查清单:**
- [ ] 同一模块的函数，同概念参数的顺序一致？
- [ ] 输出参数不在输入参数前面？

---

## F23: 不用输出参数

**级别: mandatory**

> 不通过修改传入参数来返回结果。返回值，别改入参。
> — Code Complete §7.5, Clean Code §3

**检查清单:**
- [ ] 没有函数通过改传入的 list/dict 来 "返回" 数据？
- [ ] 需要返回多值用 Result type / tuple / data class？

---

## F24: 错误返回统一

**级别: strongly-preferred**

> 一个模块的错误处理方式统一。要么全抛异常，要么全返 Result。不同时混用。
> — Code Complete §8.4

**检查清单:**
- [ ] 同一个模块不用 "抛异常" 和 "返回错误码" 混着来？
- [ ] 错误信息对调用者是可操作的 (不是 "Error: something went wrong")

---



