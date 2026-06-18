# CLI 工具参考

## 1. trivy

**用途**：依赖漏洞扫描（维度 0）+ 密钥/配置扫描（维度 F）
**安装**：`winget install AquaSecurity.Trivy` 或 `brew install trivy`
**版本**：≥0.50

### 维度 0 命令

```bash
# 漏洞扫描（仅严重+高危，加速扫描）
trivy fs --scanners vuln --severity HIGH,CRITICAL \
  --format json --output .scratch/convolution-audit/evidence/trivy-vuln.json .

# 许可证扫描
trivy fs --scanners license \
  --format json --output .scratch/convolution-audit/evidence/trivy-license.json .
```

### 维度 F 命令

```bash
# 漏洞 + 密钥扫描
trivy fs --scanners vuln,secret --severity HIGH,CRITICAL \
  --format json --output .scratch/convolution-audit/evidence/trivy-security.json .
```

### 输出摘要提取

```bash
# CVE 数量统计
cat .scratch/convolution-audit/evidence/trivy-vuln.json | \
  jq '[.Results[].Vulnerabilities[] | {id: .VulnerabilityID, severity: .Severity, pkg: .PkgName}] | group_by(.severity) | map({severity: .[0].severity, count: length})'

# 许可证冲突
cat .scratch/convolution-audit/evidence/trivy-license.json | \
  jq '[.Results[].Licenses[] | select(.Severity == "CRITICAL" or .Severity == "HIGH")] | group_by(.Category) | map({category: .[0].Category, count: length})'
```

### 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 扫描超时 | 项目太大 | 排除 node_modules：`--skip-dirs node_modules` |
| 无结果 | 无标准包管理器文件 | 标记盲区"无依赖声明文件" |
| JSON 解析失败 | 旧版 trivy | 升级到 ≥0.50 |

---

## 2. gitleaks

**用途**：密钥泄露扫描（维度 F）
**安装**：`winget install Gitleaks.Gitleaks` 或 `brew install gitleaks`
**版本**：≥8.0

### 命令

```bash
gitleaks detect --no-git --format json \
  --report-path .scratch/convolution-audit/evidence/gitleaks.json
```

### 输出摘要提取

```bash
# 统计
cat .scratch/convolution-audit/evidence/gitleaks.json | \
  jq '{total: length, files: [.[].File] | unique, rules: [.[].RuleID] | unique | length}'
```

### 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 大量误报 | 测试文件含假密钥 | 记录为"疑似泄露"，LLM agent 二次判定 |
| `--no-git` 不可用 | 非 git 仓库 | 标记盲区"gitleaks 需 git 仓库" |
| 找不到报告文件 | 无发现时 gitleaks 不输出 | 记录"gitleaks 无发现" |

---

## 3. hadolint

**用途**：Dockerfile 最佳实践检查（维度 K）
**安装**：`winget install hadolint.hadolint` 或 `brew install hadolint`
**版本**：≥2.0

### 命令

```bash
# 扫描所有 Dockerfile
hadolint Dockerfile* 2>&1 | tee .scratch/convolution-audit/evidence/hadolint.txt

# 指定单个文件
hadolint Dockerfile --format json
```

### 输出摘要提取

```bash
# 规则 ID + 级别统计
cat .scratch/convolution-audit/evidence/hadolint.txt | \
  grep -oP '(DL\d+|SC\d+)' | sort | uniq -c | sort -rn | head -10
```

### 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 无 Dockerfile | K 维度不应触发 | 检查环境检测逻辑 |
| 扫描无输出 | Dockerfile 无违规 | 记录"hadolint 通过，无违规" |
| 版本过旧 | winget 非最新 | 忽略规则号，只看违规内容 |

---

## 4. cargo-clippy

**用途**：Rust lint 检查（维度 L）
**安装**：通过 Rust 工具链 `rustup component add clippy` 或 `cargo install clippy`
**版本**：≥0.1

### 命令

```bash
# 全量 lint（允许所有级别）
cargo clippy -- -W clippy::all 2>&1 | tee .scratch/convolution-audit/evidence/clippy.txt

# 重点关注 unsafe/正确性相关的 lint
cargo clippy -- -W clippy::all -W clippy::correctness -W clippy::suspicious 2>&1 | \
  tee .scratch/convolution-audit/evidence/clippy.txt
```

### 输出摘要提取

```bash
# 按严重度统计
cat .scratch/convolution-audit/evidence/clippy.txt | \
  grep -oP '(warning|error)\[.*?\]' | sort | uniq -c | sort -rn
```

### 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| cargo 命令找不到 | Rust 未安装 | 标记盲区"Rust 工具链不可用" |
| clippy 组件未安装 | 最小安装 | `rustup component add clippy` |
| 无 Cargo.toml | 非 Rust 项目 | 检查环境检测逻辑 |

---

## 5. cargo-geiger

**用途**：Rust unsafe 代码统计（维度 L）
**安装**：`cargo install cargo-geiger`
**版本**：≥0.11

### 命令

```bash
# 完整扫描
cargo geiger 2>&1 | tee .scratch/convolution-audit/evidence/cargo-geiger.txt

# 仅计数（更快）
cargo geiger --counts-only 2>&1 | tee .scratch/convolution-audit/evidence/cargo-geiger.txt
```

### 输出摘要提取

```bash
# 提取 unsafe 计数行
cat .scratch/convolution-audit/evidence/cargo-geiger.txt | \
  grep -E '(unsafe|ffi|raw pointer)' | head -20
```

### 常见问题

| 问题 | 原因 | 解决 |
|------|------|------|
| 编译错误 | 项目需要特殊构建配置 | 标记盲区"项目无法编译，geiger 不可用" |
| 输出过长 | 大量依赖 | 用 `--counts-only` 或仅扫描项目 crate |
| 安装失败 | GitHub 被墙 | 使用 WinGet 或镜像源 |

---

## 工具可用性检查脚本

```bash
#!/bin/bash
# verify-tools.sh — 卷积审计 CLI 工具检查

echo "=== 卷积审计 CLI 工具检查 ==="
echo ""

check_tool() {
    local name=$1
    local cmd=$2
    local required=$3
    
    if which "$cmd" > /dev/null 2>&1; then
        echo "  ✓ $name: $(which $cmd)"
    else
        if [ "$required" = "required" ]; then
            echo "  ✗ $name: 未找到（必须）"
            echo "    安装: ${4:-请参考 references/cli-tools.md}"
        else
            echo "  ○ $name: 未找到（条件触发，可跳过）"
        fi
    fi
}

check_tool "trivy" "trivy" "required"
check_tool "gitleaks" "gitleaks" "required"
check_tool "hadolint" "hadolint" "optional"
check_tool "cargo-clippy" "cargo-clippy" "optional"
check_tool "cargo-geiger" "cargo-geiger" "optional"

echo ""
echo "必须工具：trivy + gitleaks 缺一不可"
echo "条件工具：hadolint（Docker 项目）/ clippy+geiger（Rust 项目）"
```
