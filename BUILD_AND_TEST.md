# 编译和测试修改后的 Codex

## 快速开始

### 1. 前置要求

确保已安装 Rust 工具链：
```bash
# 安装 Rust（如果尚未安装）
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y
source "$HOME/.cargo/env"

# 安装必要组件
rustup component add rustfmt clippy

# 安装辅助工具
cargo install --locked just cargo-nextest
```

### 2. 编译项目

```bash
# 进入 Rust 工作区
cd codex-rs

# 编译整个项目
cargo build

# 或者只编译修改的包
cargo build -p codex-core
cargo build -p codex-model-provider-info
```

**Windows PowerShell：**
```powershell
cd codex-rs
cargo build
```

### 3. 运行测试

```bash
# 运行所有测试
just test

# 或使用 cargo 直接运行
cargo test

# 只测试特定包
cargo test -p codex-core
cargo test -p codex-model-provider-info
```

---

## 测试修改的重连逻辑

### 方法 1：构建并运行

```bash
cd codex-rs

# 构建 release 版本
cargo build --release --bin codex

# 设置环境变量并运行
export CODEX_STREAM_MAX_RETRIES=3
./target/release/codex
```

### 方法 2：开发模式运行

```bash
cd codex-rs

# 直接运行开发版本
CODEX_STREAM_MAX_RETRIES=3 cargo run --bin codex -- "test prompt"
```

### 方法 3：运行相关测试

```bash
cd codex-rs

# WebSocket 降级测试
cargo test -p codex-core --test suite websocket_fallback

# 流错误处理测试
cargo test -p codex-core --test suite stream_error_allows_next_turn

# 所有 core 测试
cargo test -p codex-core
```

---

## 验证修改

### 验证 1：环境变量生效

```bash
# 默认 5 次重试
cargo run --bin codex

# 设置为 3 次
CODEX_STREAM_MAX_RETRIES=3 cargo run --bin codex

# Windows PowerShell
$env:CODEX_STREAM_MAX_RETRIES=3
cargo run --bin codex
```

### 验证 2：固定 1 秒延迟

启用日志观察重试间隔：
```bash
RUST_LOG=warn cargo run --bin codex

# 应该看到类似日志：
# "stream disconnected - retrying sampling request (1/5 in 1s)..."
# "stream disconnected - retrying sampling request (2/5 in 1s)..."
```

### 验证 3：实时显示通知

运行后观察是否显示：
```
Reconnecting... 1/5
Reconnecting... 2/5
Reconnecting... 3/5
```

✅ 第 1 次重试就显示（不再静默）

---

## 代码检查

```bash
cd codex-rs

# 格式化代码
just fmt

# Clippy 检查
just fix -p codex-core
just fix -p codex-model-provider-info
```

---

## 常见问题

### Q: cargo 命令找不到
```bash
# 安装 Rust
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source "$HOME/.cargo/env"
```

### Q: 如何只编译不运行？
```bash
cargo build -p codex-core
```

### Q: 如何清理构建？
```bash
cargo clean
```

---

**最后更新：** 2026-06-23
