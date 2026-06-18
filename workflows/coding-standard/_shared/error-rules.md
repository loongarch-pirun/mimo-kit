# 错误处理规则

程序不该无声地坏。来源: Code Complete §8, Pragmatic Programmer §4-5, Clean Code §7

---

## E02: 捕获最窄类型

**级别: mandatory | 源: Code Complete §8.3**

```
❌ catch (Exception e)     — 连 NullPointerException 一起吞
❌ except Exception:       — 任何异常都进来
✅ catch (IOException e)   — 只抓能处理的
```

---

## E03: 错误信息可操作

**级别: mandatory | 源: Pragmatic Programmer §4**

错误信息告诉调者 "怎么了" 和 "你可以怎么办"。

```
❌ "Error: Invalid input"
❌ "Something went wrong"
✅ "Invalid phone number: '0000' — expects E.164 format (+8613800138000)"
```

---

## E04: 不返回 null

**级别: mandatory | 源: Code Complete §8.3, Clean Code §7**

null 传出去就是把炸弹交给调者。调者总会忘判。

```
❌ return null;                             // 给调用方埋了颗雷
✅ return Optional.empty();                 // Java
✅ return None;                             // Rust
✅ return [];                               // 空集合
✅ raise UserNotFound(...);                 // 找不到是真的意外
```

---

## E05: 不在错误码和异常间混用

**级别: mandatory | 源: Code Complete §8.4**

一个模块的错误处理方式统一。

```
❌ funcA() 抛异常, funcB() 返回 -1, funcC() 返回 nil
✅ 模块一致: 全部抛异常，或全部返回 Result type。
```

---

## E07: 早失败

**级别: mandatory | 源: Pragmatic Programmer §4**

非法状态尽早发现——不在 5 个调用栈后才发现传了个 null。

```
❌ 函数中间赋值给某字段 → 10 分钟后抛 NPE
✅ 第 2 行判空 → 当即抛 "user must not be null"
```

---

## E08: 恢复或传播——不粉饰

**级别: mandatory | 源: Code Complete §8.3**

要抓只能真处理。不然就传播上去，不要抓了包一层假错误。

```
❌ catch (IOException) { return "file error"; }  // 把真错误丢了
✅ catch (IOException e) { throw new StorageException(e); }  // 保留根因
✅ 重试 → 仍失败 → propagate
```

---

## E09: 资源清理在 finally

**级别: mandatory | 源: Code Complete §8.3, Pragmatic Programmer §5**

```
✅ 文件/连接/锁 — 在 finally/defer/try-with-resources 里关
✅ 不在 finalize/__del__/Dispose 里做关键清理 — 时间不确定
```

---

## E10: 不被异常破坏状态

**级别: mandatory | 源: Code Complete §8.3**

中间抛异常了，对象该留在有效状态。没有 "一半改了、一半没改"。

```
✅ 修改前先验证 → 在 try 里做 — 失败回滚
✅ 不可变对象天然免疫
```
