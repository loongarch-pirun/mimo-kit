# 命名编码示例

## 变量说清意图 (N01)

### ❌ 反例
```typescript
const d = Date.now();                    // 什么的时间?
const tmp = users.filter(u => u.active);  // "临时" 一存就是三个月
const data = await fetchOrders();         // 什么数据?
```

### ✅ 正例
```typescript
const requestReceivedAt = Date.now();
const activeUsers = users.filter(u => u.active);
const pendingOrders = await fetchPendingOrders();
```

---

## 领域名跟业务一致 (N06)

### ❌ 反例
```
数据库表名:  t_cust_master
代码类名:    CustomerData
API 字段:    custInfo
业务说:      "会员"
```
一个概念三个名字 — 每次开会都要翻译。

### ✅ 正例
```
业务:  Member
代码:  Member (不是 Customer, User, Account)
建表:  members (跟代码一致)
字段:  member (统一命名)
```

---

## 同概念同名 (N08)

### ❌ 反例 (同一代码库)
```python
# users/dao.py
def fetch_user(id): ...

# orders/repository.py  
def get_user(id): ...

# notifications/service.py
def retrieve_user(id): ...

# admin/controller.py
def load_user(id): ...
```
四个动词——同一个操作 (查一个用户)。选一个单词定下来用到底。

### ✅ 正例
```python
# 项目规则: "从数据库获取单条记录" = find
def find_user(id): ...
def find_order(id): ...
def find_notification_settings(user_id): ...
```

---

## 擦除实现细节 (N09)

### ❌ 反例
```
userList       → 名字说它是 List，换了 Set 名字就撒谎了
userArray      → 名字说实现
userHashMap    → 名字说数据结构和实现两件事
userJSON       → 名字说序列化格式
```

### ✅ 正例
```
users          → 不管存成 List/Set/Array，名字不变
userMap        → 说用途 (映射查找)，不说实现
userLookup     → 同上
```
