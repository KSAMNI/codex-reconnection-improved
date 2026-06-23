# Codex 重连逻辑修改说明

## 修改概述

本次修改优化了 Codex CLI 的网络重连体验，主要包括三个方面：

1. **通过环境变量控制重试次数**
2. **固定重试延迟为 1 秒**
3. **移除静默逻辑，所有重试都实时显示**

---

## 修改详情

### 1. 环境变量支持 (CODEX_STREAM_MAX_RETRIES)

**文件：** `codex-rs/model-provider-info/src/lib.rs`

**变更：**
- 新增常量：`const ENV_STREAM_MAX_RETRIES: &str = "CODEX_STREAM_MAX_RETRIES"`
- 修改 `stream_max_retries()` 方法，优先读取环境变量

**使用方法：**
```bash
# 设置重试次数为 10 次
export CODEX_STREAM_MAX_RETRIES=10

# 或在 Windows PowerShell 中
$env:CODEX_STREAM_MAX_RETRIES=10

# 然后运行 codex
codex
```

**优先级：**
1. 环境变量 `CODEX_STREAM_MAX_RETRIES`（最高优先级）
2. 配置文件中的 `stream_max_retries` 设置
3. 默认值 5 次

**上限：** 最大 100 次（硬性上限，防止无限重试）

---

### 2. 固定延迟策略

**文件：** `codex-rs/core/src/responses_retry.rs`

**变更前（指数退避）：**
- 第 1 次重试：200ms
- 第 2 次重试：400ms
- 第 3 次重试：800ms
- 第 4 次重试：1600ms
- 第 5 次重试：3200ms

**变更后（固定延迟）：**
- 所有重试：**1000ms (1秒)**

**代码变更：**
```rust
// 移除了对 backoff() 函数的依赖
// 新增常量
const RETRY_DELAY_MS: u64 = 1000;

// 使用固定延迟
let delay = Duration::from_millis(RETRY_DELAY_MS);
```

**优势：**
- 更可预测的重连时间
- 快速失败，避免长时间等待
- 用户体验更一致

---

### 3. 移除静默逻辑

**文件：** `codex-rs/core/src/responses_retry.rs`

**变更前：**
- Release 模式下，第 1 次 WebSocket 重试**静默**（不显示通知）
- Debug 模式下，所有重试都显示

**变更后：**
- **所有重试都显示通知**
- 无论 Debug 或 Release 模式
- 无论 WebSocket 或 HTTPS 传输

**通知格式：**
```
Reconnecting... 1/5
Reconnecting... 2/5
...
```

**优势：**
- 用户可以实时了解网络状态
- 不会出现"卡住"的错觉
- 更透明的错误处理

---

## 测试建议

### 1. 测试环境变量
```bash
# 测试设置为 3 次重试
export CODEX_STREAM_MAX_RETRIES=3
codex

# 观察是否在 3 次重试后失败或降级到 HTTPS
```

### 2. 测试固定延迟
- 断开网络连接
- 运行 codex 命令
- 观察每次重试之间是否间隔 1 秒

### 3. 测试实时显示
- 在不稳定的网络环境下运行
- 确认每次重试都显示 "Reconnecting..." 消息
- 确认第 1 次重试就显示通知

### 4. 测试降级逻辑
- 设置较少的重试次数（如 2 次）
- 观察达到最大重试次数后是否显示 "Falling back from WebSockets to HTTPS transport"

---

## 兼容性说明

### 向后兼容
- 所有现有配置文件继续有效
- 未设置环境变量时使用默认值
- 配置文件中的 `stream_max_retries` 仍然生效

### 破坏性变更
- 延迟策略从指数退避改为固定 1 秒
- Release 模式下第 1 次重试不再静默

---

## 配置示例

### 通过环境变量配置
```bash
# Linux/macOS
export CODEX_STREAM_MAX_RETRIES=10

# Windows PowerShell
$env:CODEX_STREAM_MAX_RETRIES=10

# Windows CMD
set CODEX_STREAM_MAX_RETRIES=10
```

### 通过配置文件配置
```toml
# config.toml
[model_provider]
stream_max_retries = 10
```

### 持久化环境变量
```bash
# Linux/macOS - 添加到 ~/.bashrc 或 ~/.zshrc
echo 'export CODEX_STREAM_MAX_RETRIES=10' >> ~/.bashrc

# Windows - 系统环境变量设置
setx CODEX_STREAM_MAX_RETRIES 10
```

---

## 相关文件清单

| 文件路径 | 修改内容 |
|---------|---------|
| `codex-rs/core/src/responses_retry.rs` | 固定延迟 + 移除静默逻辑 |
| `codex-rs/model-provider-info/src/lib.rs` | 环境变量支持 |

---

**修改日期：** 2026-06-23
