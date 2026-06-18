# 模块级编码示例

## 依赖反转 (M04)

### ❌ 反例
```typescript
// domain/order-service.ts
import { MySQLConnection } from "../infra/mysql-connection"; // ❌ 领域依赖基础设施

class OrderService {
    private db: MySQLConnection;
    
    placeOrder(order: Order) {
        this.db.execute("INSERT INTO orders ..."); // ❌ 领域代码知道 SQL
    }
}
```

### ✅ 正例
```typescript
// domain/order-service.ts
interface OrderRepository {       // 抽象在领域层定义
    save(order: Order): void;
}

class OrderService {
    constructor(private repo: OrderRepository) {}  // 依赖抽象
    
    placeOrder(order: Order) {
        this.repo.save(order);    // 领域代码不接触基础设施细节
    }
}

// infra/mysql-order-repository.ts
import { OrderRepository } from "../domain/order-service";  // 基础设施依赖领域
class MySQLOrderRepository implements OrderRepository { ... }
```

---

## 聚合根 (M09)

### ❌ 反例
```java
// 直接穿透聚合边界
OrderLine line = order.getLines().get(2);   // 绕过 Order
line.setQuantity(10);                       // 改内部对象
lineRepository.save(line);                  // 单独持久化内部对象
```

### ✅ 正例
```java
// 只能通过聚合根操作
order.adjustLineQuantity(lineId, 10);       // Order 自己改自己内部
orderRepository.save(order);                // 整体持久化
```

---

## 值对象 (M11)

### ❌ 反例
```python
class User:
    id: str            # Entity
    email: str         # ❌ 裸字符串 —— 任何调用方需要知道什么是合法 email
    balance: float     # ❌ 裸浮点 —— 是人民币还是美元? 含不含税?
    address: str       # ❌ 裸字符串 —— 格式不确定
```

### ✅ 正例
```python
class Email:
    value: str
    def __post_init__(self): self._validate()
    # equals, hashCode, toString — 值语义

class Money:
    amount: Decimal
    currency: Currency
    # 算术方法返回新 Money, 不修改原对象

class User:
    id: UserId
    email: Email       # 自解释、自验证
    balance: Money      # 含单位、不可变
```

---

## 信息隐藏 (M06)

### ❌ 反例
```go
// 暴露内部结构
func (s *UserService) GetDB() *sql.DB {  // ❌ 调者能直接查库
    return s.db
}

// 调者容易跳过业务逻辑直接查
rows, _ := userService.GetDB().Query("SELECT * FROM users")
```

### ✅ 正例
```go
// 只暴露行为
func (s *UserService) FindActiveUsers() ([]User, error) {
    return s.repo.FindByStatus("active")
}
// 调者无法绕过业务逻辑 —— 查库必须走 UserService
```
