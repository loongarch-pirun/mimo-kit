# 命名规则

代码命名是跟未来的自己 (和队友) 的对话。来源: Code Complete §11, DDD Ubiquitous Language, Clean Code §2

---

## N02: 函数名是动词

**级别: mandatory | 源: Code Complete §7.3**

函数做动作——命名说了它做哪个动作。

```
❌ user(), order(), items()          — 名词，读作"我是 User"
✅ getUser(), createOrder(), listItems()  — 动词，读作"我做这件事"
```

函数名格式: `动词 + 名词` 或 `动词 + 形容词 + 名词`。

---

## N04: 类和接口是名词

**级别: mandatory | 源: Clean Code §2**

```
❌ ProcessData, ManageUser, HandleOrder   — 太模糊
✅ UserRepository, EmailService, OrderAggregate
```

---

## N06: 领域名跟业务一致

**级别: mandatory | 源: DDD Ubiquitous Language**

业务叫 "会员" → 代码叫 `Member`。业务说 "下单" → 代码叫 `placeOrder`。

```
❌ 业务叫 Customer → 代码叫 User
❌ 业务叫 Shipment → 代码叫 DeliveryData
✓ 哪个团队、哪个会议室、哪个白板上写什么——代码里就叫什么
```

---

## N07: 不缩写

**级别: strongly-preferred | 源: Code Complete §11.3**

```
❌ usr, addr, calAmt, idx, ctrl
✅ user, address, calculateAmount, index, controller
```

例外 (全行业统一的): `id`, `url`, `json`, `http`, `xml`, `db`, `io`, `api`

---

## N08: 同概念同名

**级别: mandatory | 源: Code Complete §11.4**

一个代码库里同一个概念用同一个词。不要这文件 `fetch`, 那文件 `retrieve`, 第三个 `get`。

```
❌ 不同模块: fetch / retrieve / get / load / read — 全指 "查数据"
✅ 一条规则: "获取" = fetch (选一个词定下来)
```

---

## N09: 擦除实现细节

**级别: strongly-preferred | 源: Code Complete §11.5**

名字不说是怎么存的、传的、序列化的。

```
❌ userList, userArray, userHashMap, userJSON
✅ users, userMap, userLookup
```

---

## N10: 命名长度按作用域定

**级别: strongly-preferred | 源: Code Complete §11.2**

作用域越大，名字越详。循环变量 i 没问题——它只活三行。全局常量必须能自解释。
