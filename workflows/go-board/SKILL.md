---
name: go-board
description: >
  需求驱动的 VM 博弈测试靶场。三阶段：和平棋盘 → 加载攻击知识库 → VM 博弈（攻守交替 + 锚定保护）。
  管线（/orchestrate）全部跑完之后，产出的成品代码扔进 VM 里真刀真枪暴打。管线是工厂，这是出厂暴击实验室。
  触发：go-board / 棋盘测试 / 上棋盘 / 需求验证
tools: Bash, Read, Write, Edit, Grep, Glob
---

# /go-board — VM 博弈测试靶场

你是围棋裁判。你的棋盘是 VM，棋子是代码和 ChaosBlade 攻击。这盘棋的规则由 PRD 决定。

## 前置条件

- PRD + SPEC 已产出（`spec/SPEC.md` 或用户指定）
- 成品代码已生成（在项目目录中）
- Vagrant + VirtualBox 已安装
- `~/.claude/vm-lab/` 目录完整

## 全局流程

```
启动 /go-board
  │
  ├── 阶段 0：和平棋盘
  │   不打，先验证"没压力时功能对不对"
  │   PRD 每句话 → 功能验收 → VM 里跑
  │   全部通过 → 进下一阶段
  │   失败 → 终止，输出哪些需求没过
  │
  ├── 阶段 1：加载攻击知识库
  │   PRD 关键词 → 命中 kb/*.yaml → 生成攻击计划
  │   导出 attack-plan.json
  │
  ├── 阶段 2：VM 博弈
  │   vagrant up → provision → 部署代码 →
  │   守夜人循环（出招 → 裁判三问 → SOS → 应手 →
  │   锚定回归 → 再出招）→ 终局
  │
  └── 输出：棋谱 + 终局报告
```

## 阶段 0：和平棋盘

```bash
# 1. 确保 VM 不运行
vagrant halt 2>/dev/null || true

# 2. 拷贝 PRD + 代码到 vm-lab 目录
# 3. 在 VM 里运行和平棋盘
vagrant up
vagrant ssh -- "bash /vagrant/peace-board.sh /vagrant/PRD.md /vagrant"

# 4. 检查结果
vagrant ssh -- "cat /vagrant/test-results/peace-board-*.json"
```

**判定**：
- PASS → 记录锚定矩阵基线 → 进阶段 1
- FAIL → 输出哪些需求没过 → 终止。回到 /implement 修。

## 阶段 1：加载攻击知识库

```bash
# 生成攻击计划（从 PRD 关键词命中知识库）
vagrant ssh -- "bash /vagrant/attack-loader.sh /vagrant/PRD.md /vagrant/../vm-lab/kb /vagrant/attack-plan.json"

# 查看攻击列表
vagrant ssh -- "cat /vagrant/attack-plan.json | jq '.attacks[] | \"\(.name) [\(.severity)]\"'"
```

**输出攻击摘要，让用户知情后进阶段 2。**

## 阶段 2：VM 博弈

```bash
# 部署项目代码到共享目录
# 启动目标服务
# 启动守夜人
vagrant ssh -- "bash /vagrant/watchman.sh"
```

### SOS 回调：Claude 的应手协议

守夜人在博弈过程中会写 SOS 文件到 `$PROJECT_DIR/sos/`。

你（Claude）需要：
1. **监听**：检查 `sos/` 目录是否有新的 `sos-round-N.json`
2. **分析**：读 SOS 内容（崩溃日志、堆栈、原因）
3. **应手**：在项目源码中修改修复，生成 patch
4. **投递**：把 patch 写入 `sos/counter-round-N.patch`
5. **继续监听**：守夜人会捡起 patch、应用、锚定回归、重测

**应手协议规则**：
- 一次只修一个问题（不顺手修"顺便看到的"）
- 修复必须最小化（解决当前崩溃即可，不过度设计）
- 修复后检查：这个修改会不会影响其它需求？（心里跑一遍锚定矩阵）
- patch 包含完整的 diff，守夜人直接 `git apply`

### 终局判断

守夜人退出后检查终局文件：

```bash
vagrant ssh -- "cat /vagrant/kifu/endgame-*.json"
```

- `result: "PASS"` → 代码不可击破。输出棋谱摘要。
- `result: "FAIL"` → 还有未解决问题。输出问题列表，触发 repair-engine。

## 调用方式

### 独立使用
```
/go-board
/go-board --platform=windows
/go-board --level=quick    # 快速模式：缩短攻击超时，减少轮次
```

## 输出

```
═══════════════════════════════════
  Go-Board 终局报告

  产品：{product_name}
  产品类型：{matched_kb}
  棋盘：Ubuntu 22.04 VM

  和平棋盘：5/5 ✓
  VM 博弈：
    总轮次：{N}
    攻击数：{M}
    修复数：{F}
    回归引入：0
    未解决问题：0

  结论：代码符合需求，压力下行为正确。
  棋谱：{kifu_path}
═══════════════════════════════════
```

## 逃生门

以下情况立即停止博弈：
- 连续 3 次修复引入新回归 → 人工介入
- 同一攻击连续 5 轮未修复 → 标记为已知弱点，跳过，继续
- VM 资源耗尽（OOM / disk full）→ 扩容后重试
- Vagrant 启动失败 → 检查环境，不阻塞管线
