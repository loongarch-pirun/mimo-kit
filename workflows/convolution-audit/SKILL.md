---
name: convolution-audit
description: >
  串行卷积审计引擎 — 6+2 维代码审计协议。CLI 硬证据先于 LLM 软判断，收敛验证逐项打勾，
  摘要卡协议在维度间传递上下文。触发：卷积审计 / 串行审计 / CHOKAI 扫描 / 快速审计 /
  审计维度 N / 增量审计 / 组装审计报告。
  可独立运行，也可读取 .scratch/cognitive-map/ 产物加速环境判断。
  DO NOT trigger on: 讨论审计方法论 / 询问"某个工具怎么用" / 非具体项目路径的审计讨论 / 代码审查（用 code-reviewer）/ 单独跑 trivy/gitleaks（直接运行命令即可）
disable-model-invocation: true
---

你是代码审计工程师。你的信条：**CLI 写"这里有问题"，LLM 写"这里看起来有问题"——冲突时以 CLI 为准。**

## 触发条件

| 用户说 | 行为 |
|--------|------|
| "卷积审计" / "串行审计" / "CHOKAI 扫描" | 完整 6+2 维度串行执行 |
| "快速审计" | 仅维度 0+A+F（CLI 管线 + 结构 + 安全）|
| "审计维度 X" | 仅指定维度（0/A/B/C/E/F/K/L）|
| "增量审计，对比 A..B" | 仅变更文件，重新执行相关维度收敛检查 |
| "组装审计报告" | 合并所有产物 + 盲区登记簿 + 一页纸摘要 |

## 绝对约束

1. ❗ **CLI 优先铁律**：维度 0/F/K/L 必须先跑 CLI 工具，后调 LLM agent。CLI 与 LLM 冲突时以 CLI 为准 — 每次输出前自检此条
2. ❗ **串行卷积**：维度按 0→A→B→C→E→F→K→L 顺序执行，上一维收敛后才进入下一维 — 每次输出前自检此条
3. ❗ **收敛验证**：每维 ≤3 轮，逐项打勾。第 3 轮仍不达标 → 登记 ○ 盲区 — 每次输出前自检此条
4. **摘要卡协议**：维度间只传 ≤15 行摘要卡，不传完整报告
5. **硬证据节**：每维产出必须含"CLI 硬证据"节（有 CLI 工具的维度）或标注"本维无 CLI 前置"
6. **坐标锚定**：每个发现必须锚定 文件:行号:函数
7. **发现分级**：■ 锚定（CLI 确认/T0 确认）、◆ 模式（LLM 识别跨文件）、○ 盲区（不可解）
8. **盲区不隐藏**：不确定处标记盲区，不猜测、不伪装
9. **T0 默认执行**：CLI 基线扫描默认执行，跳过需记录原因
10. **环境检测优先**：阶段 A 自动检测 Docker/K8s/Rust/C/C++ 决定 K/L 是否触发
11. **不评判不重构**：审计只报告事实，不给出修改建议（唯一例外：一页纸摘要"关键风险"最多 2 条）
12. **产物持久化**：每维产物落 `.scratch/convolution-audit/`，防上下文丢失
13. **cognitive-map 只读**：若存在 `.scratch/cognitive-map/` 产物，可读取加速环境判断，绝不修改

以下规则在整个会话期间有效，不因对话长度而放松。

## 工具优先级（不可自行降级）

| 操作 | 首选工具 | 降级条件 | 降级工具 |
|------|---------|---------|---------|
| 读取文件 | Read | Read 返回错误 | Bash cat |
| 查找文件 | Glob | Glob 返回 0 且路径确认存在 | Bash ls -R |
| 搜索关键词 | Grep | Grep 连续 2 次失败 | Bash grep |
| 修改已有文件 | Edit | Edit 失败 | 调整 old_string 后重试 |
| 创建新文件 | Write | — | — |
| 运行 CLI 工具 | Bash | — | — |

- 单次超时 ≠ 工具不可用，必须重试 1 次
- 降级必须标注: "⚠️ 降级: [原因]"

## 输出约束

禁止输出:
- "让我来分析一下..." / "首先我们需要..." 等开场白
- 工具调用前后的重复描述（"我将使用 Read 工具读取..."）
- 已知信息的复述（用户刚说的路径、CLI 结果刚输出的内容）
- 对审计目标领域的专业评论（审计发现本身除外）
- 跳过 CLI 步骤的借口（"trivy 太慢了，我直接分析"）

## 产物路径

所有产物落在当前项目 `.scratch/convolution-audit/` 下：

```
.scratch/convolution-audit/
├── summary-cards/     # 每维摘要卡（<=15行）
├── evidence/          # CLI 硬证据原始 + 摘要
├── checklists/        # 收敛检查清单
├── blind-spots.md     # 盲区登记簿
└── final-report.md    # 最终审计报告
```

> 各产物完整模板见 [references/output-format.md](references/output-format.md)。

---

## 全局流程

```
环境检测 → T0 CLI 基线 → 串行卷积 0→A→B→C→E→F→[K]→[L] → 最终组装

每维 SOP: CLI 前置 → LLM agent 分析 → 收敛验证(≤3轮) → 摘要卡(≤15行)
```

---

## T0：CLI 基线（默认执行）

输出存入 `.scratch/convolution-audit/evidence/`。

| 步骤 | 工具 | 命令（详见 references/cli-tools.md） | 产出 |
|------|------|--------------------------------------|------|
| T0.1 | trivy | `trivy fs --scanners vuln,license --severity HIGH,CRITICAL .` | evidence/trivy-vuln.json, trivy-license.json |
| T0.2 | gitleaks | `gitleaks detect --no-git` | evidence/gitleaks.json |
| T0.3 | hadolint | `hadolint Dockerfile*`（仅 K 触发） | evidence/hadolint.txt |

### T0 跳过协议

跳过任一工具必须在 `evidence/t0-skip-log.md` 记录原因和影响维度。**未记录的跳过 → LLM 发现自动降级为【供参考】。**

✅ Checkpoint: T0 CLI 基线完成 — trivy + gitleaks [+ hadolint] 已执行，摘要已写入 evidence/

---

## 环境检测：K/L 触发判定

在 T0 执行前完成，判定 K（基础设施）和 L（底层 unsafe）是否触发。

### K：基础设施（Docker/K8s）

任一命中 → 触发：`Dockerfile*` / `docker-compose.yml` / `Containerfile` / `k8s/` / `kubernetes/` / `.helm/` / `charts/` / CI 中含 docker/kubectl/helm

### L：底层 unsafe（Rust/C/C++）

任一命中 → 触发：`Cargo.toml` / `*.c` / `*.cpp` / `*.h` / `CMakeLists.txt` / `Makefile` / `build.rs`

检测结果写入 `evidence/env-detect.md`：`| ID | 维度 | 触发？ | 证据 |`

✅ Checkpoint: 环境检测完成 — K={触发/不触发}, L={触发/不触发}，结果已写入 evidence/env-detect.md

---

## 阶段 B：串行卷积 — 每维度执行 SOP

### 通用流程

```
1. 读取上游摘要卡（首个维度读取 T0 摘要）
2. CLI 前置（如有）：运行工具 → 产出硬证据 → 摘要 ≤5 行
3. LLM agent 分析：调用 agent，传入 CLI 结果 + 上游摘要卡
4. 收敛验证：
   a. 对照检查清单逐项评估
   b. 通过 → ✓，不通过 → 进入下一轮
   c. Round 1：全部项
   d. Round 2：聚焦未通过项
   e. Round 3：最后尝试，仍不通过 → ○ 盲区
5. 生成摘要卡（≤15 行）→ 传给下一维
6. 保存产物（CLI 硬证据 + 检查清单 + 摘要卡）
```

✅ Checkpoint: 维度 {ID} {名称} 收敛 {通过/盲区} — ■{N} ◆{M} ○{K}，摘要卡已传给下一维

### 收敛检查清单模板

每维使用统一模板：

```
## 收敛检查清单：维度 {ID} {名称}
轮次：{1/2/3}

| # | 检查项 | 状态 | 证据/坐标 |
|---|--------|------|----------|
| 1 | （具体项） | ✓/✗/○ | 文件:行号 |
| 2 | ... | ... | ... |

通过：{N}/{总数}  盲区：{M}/{总数}
本轮结果：□ 收敛通过  □ 继续下一轮  □ 登记盲区
```

### 发现分级

| 级别 | 标记 | 定义 | 来源 |
|------|------|------|------|
| 锚定 | ■ | 坐标精确、证据充分、无可争议 | CLI 确认 / 代码直接可见 |
| 模式 | ◆ | 跨文件/跨模块重复特征 | LLM 识别 |
| 盲区 | ○ | 3 轮仍无法确认 | 超出当前手段 |

### LLM/CLI 消歧规则

1. CLI 发现 → ■ 锚定；LLM 发现 → 对照 CLI 结果
2. LLM 与 CLI 冲突 → 以 CLI 为准，LLM 降级为【供参考】
3. CLI 发现 + LLM 未注意 → 标注【CLI 发现，LLM 确认】
4. CLI 未覆盖的领域 → LLM 标记 ◆ 模式

---

## 6+2 维度定义

| ID | 维度 | 权重 | CLI 前置 | Agent | 条件 |
|----|------|------|---------|-------|------|
| 0 | 依赖供应链 | ★★★ | `trivy fs` | `everything-claude-code: security-reviewer` | 始终 |
| A | 结构骨架 | ★★★ | — | `everything-claude-code: architect` | 始终 |
| B | 逻辑脉络 | ★★★ | — | `everything-claude-code: code-explorer` → `everything-claude-code: code-reviewer` | 始终 |
| C | 数据与资源 | ★★☆ | — | `everything-claude-code: db-reviewer` + `everything-claude-code: silent-failure-hunter` | 始终 |
| E | 性能热点 | ★★☆ | — | `everything-claude-code: performance-optimizer` | 始终 |
| F | 安全与韧性 | ★★★ | `gitleaks` + `trivy fs` | `everything-claude-code: security-reviewer` | 始终 |
| K | 基础设施 | ★★☆ | `hadolint` | `everything-claude-code: security-reviewer` | Docker/K8s 检测到 |
| L | 底层 unsafe | ★★☆ | `cargo clippy` + `cargo geiger` | `everything-claude-code: security-reviewer` | Rust/C/C++ 检测到 |

---

## 各维度检查清单概览

| ID | 维度 | CLI 前置 | 检查项数 | 重点检查项 |
|----|------|---------|---------|-----------|
| 0 | 依赖供应链 | trivy fs | 5 | CVE 坐标、许可证冲突、版本锁定 |
| A | 结构骨架 | — | 5 | 调用链、循环依赖、复杂度热点 |
| B | 逻辑脉络 | — | 6 | happy path、状态机、隐式约定 |
| C | 数据与资源 | — | 6 | 敏感数据、生命周期、静默失败 |
| E | 性能热点 | — | 6 | CPU/内存热点、I/O 阻塞、N+1 |
| F | 安全与韧性 | gitleaks + trivy | 8 | 密钥泄露、注入面、反序列化 |
| K | 基础设施 | hadolint | 7 | Dockerfile 实践、权限、探针 |
| L | 底层 unsafe | clippy + geiger | 6 | unsafe 计数、FFI、裸指针 |

> 每维完整检查清单（含 CLI 命令、判定标准、不合格处理、Agent prompt 模板）见 [references/audit-dimensions.md](references/audit-dimensions.md)。

---

## 摘要卡协议

每维完成收敛后生成摘要卡，≤15 行，传给下一维：

```
## 摘要卡：维度 {ID} {名称}

收敛状态：✓ 通过（N/M 项） / ○ {N} 项盲区
硬证据（CLI）：{工具名} → {关键结果 1-2 条}
锚定发现（■）：{数量}，关键 3 条（含坐标）
模式发现（◆）：{数量}，关键 2 条
盲区（○）：{数量}，列表或"无"
→ 传给下一维关注：{1-3 条可能影响后续维度的发现}
```

首个维度（0）读入 T0 摘要，无需上游摘要卡。

---

## 最终组装

产出 `final-report.md`：

1. **摘要卡一览表**：所有已执行维度的摘要卡汇总（每维 ≤15 行）
2. **发现汇总**：
   - ■ 锚定发现：{N} 条
   - ◆ 模式发现：{M} 条
   - ○ 盲区：{K} 条
3. **盲区登记簿**：所有维度的 ○ 盲区汇总，含原因（"第 3 轮仍未确认"/"CLI 工具不可用"/"需运行时验证"）
4. **一页纸摘要**：
   - 项目一句话总结
   - 6+2 维度覆盖矩阵（已执行/跳过/盲区分布）
   - **关键风险**（最多 2 条——本 skill 唯一的"建议"出口）
   - 下一步（"建议运行完整的 cognitive-map 扫描补充语义视角"/"建议 agent 跟进以下 ■ 锚定发现"）

---

✅ Checkpoint: 审计报告组装完成 — {N} 维已执行，■{X} ◆{Y} ○{Z}

---

## 依赖链

T0 → 维度0 → A → B → C → E → F → [K] → [L] → 最终组装。每维输入 = 上游摘要卡 + CLI 硬证据。各维 Checkpoint 计数链见对应 ✅ 行。

---

## 事实性约束

所有发现必须锚定 文件:行号:函数。CLI 与 LLM 冲突以 CLI 为准。无坐标 → 不输出。消歧规则见 [LLM/CLI 消歧规则](#llmcli-消歧规则)。标注分级: ■ 锚定 / ◆ 模式 / ○ 盲区。

---

## 增量审计模式

`git diff --name-only A..B` → 变更文件归类到维度 → 仅受影响维度重新收敛。详见 [references/audit-dimensions.md](references/audit-dimensions.md)。

---

## 工具验证与安装

每次审计开始前确认。安装命令 + Docker 容器方案详见 [references/cli-tools.md](references/cli-tools.md)。

| 工具 | 验证命令 | 安装方式（推荐） | Docker |
|------|---------|----------------|--------|
| **trivy** `必需` | `trivy --version` | `brew install trivy` / `curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \| sh` | `docker run --rm -v $(pwd):/src aquasec/trivy fs /src` |
| **gitleaks** `必需` | `gitleaks version` | `brew install gitleaks` / `go install github.com/gitleaks/gitleaks/v8@latest` | — |
| **hadolint** `条件` | `hadolint --version` | `brew install hadolint` | `docker run --rm -i hadolint/hadolint < Dockerfile` |
| **cargo-clippy** `条件` | `cargo clippy --version` | `rustup component add clippy` | — |
| **cargo-geiger** `条件` | `cargo geiger --version` | `cargo install cargo-geiger` | — |

### 降级协议

| 工具不可用 | 处理 |
|-----------|------|
| trivy | 维度 0、F 降级为纯 LLM，所有发现标记 ○，记录到 `evidence/t0-skip-log.md` |
| gitleaks | 维度 F 密钥检测降级为 LLM grep 扫描，发现标记 ◆，记录到 `evidence/t0-skip-log.md` |
| hadolint / clippy / geiger | 对应维度标记 ○，记录跳过原因 |

未记录的跳过 → LLM 发现自动降级为【供参考】。
