# 函数级编码示例

## 单一职责 (F01)

### ❌ 反例
```python
def update_profile_and_notify(user_id, name, email, avatar_url):
    user = db.query(f"SELECT * FROM users WHERE id = {user_id}")
    user['name'] = name
    user['email'] = email
    db.execute(f"UPDATE users SET name='{name}', email='{email}' WHERE id={user_id}")
    if old_email != email:
        smtp.send(email, "Your email has been updated")
    cache.delete(f"user:{user_id}")
    log.info(f"Profile updated for {user_id}")
```
问题: 查库、改字段、发邮件、清缓存、写日志——混在一个函数。

### ✅ 正例
```python
def update_profile(user_id, profile_update):
    user = user_repository.find_by_id(user_id)
    previous_email = user.email
    user.apply(profile_update)
    user_repository.save(user)
    if user.email_changed_from(previous_email):
        notification_service.send_email_changed(user)
    cache.invalidate_user(user_id)
```
每个操作是它自己的方法。`update_profile` 只管编排。

---

## 命名不撒谎 (F05)

### ❌ 反例
```typescript
function getUser(id: string): User | null {
    auditLog.record(`user ${id} queried`);  // 副作用!
    return db.users.findById(id);
}
```
名字是 `getUser` —— 调者以为纯查询，结果附带审计日志。

### ✅ 正例
```typescript
function getUser(id: string): User | undefined {
    return db.users.findById(id);
}
// 审计在调用层显式做:
function getUserWithAudit(id: string): User | undefined {
    const user = getUser(id);
    auditLog.record(`user ${id} queried`);
    return user;
}
```

---

## 不用否定条件 (F07)

### ❌ 反例
```python
if not is_invalid(user):
    proceed()
```
读作 "如果不-不-合法的用户，继续"——大脑要转两层。

### ✅ 正例
```python
if is_valid(user):
    proceed()
```

---

## CQS 分离 (F11)

### ❌ 反例
```java
public User getOrCreateUser(String email) {
    User user = repo.findByEmail(email);
    if (user == null) {
        user = new User(email);
        repo.save(user);
    }
    return user;
}
```
读这个函数的人无法知道"它只是查，还是可能写"——名字两可。

### ✅ 正例
```java
// 命令 — 确保存在
public User ensureUserExists(String email) {
    return repo.findByEmail(email)
        .orElseGet(() -> repo.save(new User(email)));
}

// 查询 — 纯获取
public Optional<User> findUser(String email) {
    return repo.findByEmail(email);
}
```

---

## 嵌套不超过 3 层 (F08)

### ❌ 反例
```javascript
function process(order) {
    if (order.isPaid) {
        if (order.hasItems) {
            for (const item of order.items) {
                if (item.isInStock) {
                    if (item.quantity <= item.stock) {
                        ship(item);
                    }
                }
            }
        }
    }
}
```

### ✅ 正例
```javascript
function process(order) {
    if (!order.isPaid) return;
    if (!order.hasItems) return;
    for (const item of order.items) {
        shipIfPossible(item);
    }
}

function shipIfPossible(item) {
    if (!item.isInStock) return;
    if (item.quantity > item.stock) return;
    ship(item);
}
```

---

## 早返回 (F13)

### ❌ 反例
```python
def calculate_discount(user, order):
    if user is not None:
        if order is not None:
            if order.total > 0:
                if user.is_premium:
                    return order.total * 0.2
                else:
                    return 0
            else:
                return 0
        else:
            return 0
    else:
        return 0
```

### ✅ 正例
```python
def calculate_discount(user, order):
    if user is None: return 0
    if order is None: return 0
    if order.total <= 0: return 0
    if not user.is_premium: return 0
    return order.total * 0.2
```

---

## 注释说为什么不是说是什么 (F18)

### ❌ 反例
```python
# Increment i by 1
i = i + 1

# Call the API
response = requests.post(url, json=data)
```

### ✅ 正例
```python
i += 1  # Every 5th item gets a separator — this tracks position

# Retry with backoff because the payment gateway returns 429
# when two requests land in the same 100ms window
response = requests.post(url, json=data)
```
