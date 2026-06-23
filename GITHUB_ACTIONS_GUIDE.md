# 使用 GitHub Actions 编译和测试

## 快速开始

### 1. 提交代码

```bash
cd I:\Code\codex

git add .
git commit -m "feat: 改进重连逻辑"
git push origin main
```

### 2. 触发构建

1. 打开 GitHub 仓库
2. 点击 **Actions** 标签
3. 选择 **Build Modified Codex**
4. 点击 **Run workflow**

### 3. 下载构建产物

等待构建完成（约 15 分钟），然后：
1. 在 Actions 页面点击完成的运行
2. 滚动到底部 **Artifacts** 部分
3. 下载对应平台的文件

## Windows 测试

```powershell
# 下载 codex-windows-x64.exe 后
$env:CODEX_STREAM_MAX_RETRIES=10
.\codex-windows-x64.exe --version
```

## 验证修改

运行后观察：
- ✅ 显示 "Reconnecting... 1/X"
- ✅ 每次间隔 1 秒
- ✅ 环境变量控制重试次数

## 故障排除

### Actions 权限
Settings > Actions > General > 启用 "Read and write permissions"

### macOS Gatekeeper
```bash
xattr -d com.apple.quarantine codex-macos-arm64
```

---

**创建日期：** 2026-06-23
