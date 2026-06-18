# Coding Standard — 项目级配置
# 放到项目根目录，本 Skill 自动读取并应用
# 不改也可以——默认全部规则生效，全部为默认严重度

version: 1

# ── 禁用规则 ──────────────────────────────────────────
# 列出要跳过的规则 ID。被禁用的规则在写代码时不检查。
# 可用 ID: F01-F25, M01-M13, N01-N10, E01-E10
disable:
  # - F02        # 示例: 不强制 ≤20 行 (项目用长函数风格)
  # - T05        # 示例: 测试友好不作为硬约束

# ── 严重度调整 ────────────────────────────────────────
# 某些规则在你的项目里没那么严重——降级。
# 可用严重度: critical / warning / suggestion
severity:
  # F02: warning          # 示例: 函数 >20 行降为 warning (不阻断)
  # M05: suggestion       # 示例: 深模块降为建议 (项目还在早期)

# ── 忽略路径 ──────────────────────────────────────────
# glob 模式——匹配的文件完全不检查。
ignore:
  - "**/*.generated.*"
  - "**/vendor/**"
  - "**/migrations/**"
  - "**/__pycache__/**"
  - "**/*.pb.go"           # protobuf 生成代码
  - "**/*.g.dart"          # Dart build_runner 输出

# ── 聚焦模式 ──────────────────────────────────────────
# 非空列表: 只检查列出的规则，其他全跳过。
# 空列表或省略: 检查所有非禁用规则。
# focus 和 disable 不能同时为非空——同时非空时两者都被忽略。
focus: []
  # - F01
  # - F04
  # - N06

# ── 抑制项 ────────────────────────────────────────────
# 特定文件/行的已知例外——"我知道这里不对，但现在不修，有理由"。
# 每项: risk (规则 ID), pattern (文件 glob), reason (为什么), expires (可选，到期日期)
suppress:
  # - risk: F01
  #   pattern: "src/legacy/order-handler.ts"
  #   reason: "迁移中——Q3 拆成 OrderService + NotificationService"
  #   expires: 2026-09-30
  #
  # - risk: M04
  #   pattern: "src/adapters/old-mysql-adapter.ts"
  #   reason: "历史债务——该 adapter 会在下个 milestone 被新实现替换"
  #   expires: 2026-07-15

# ── 自定义风险 ──────────────────────────────────────────
# 项目独有的规则——12 本经典没覆盖，但你项目需要。
# 每条: id (C1..CN), description, check (人工可读的检查项), severity, source (可选)
custom_rules:
  # - id: C1
  #   description: "所有 API 端点必须有 rate-limit 注释"
  #   check: "每个 @Route 注解上面是否有 // @RateLimit(N) 注释？"
  #   severity: warning
  #   source: "项目安全规范 v2.1"
