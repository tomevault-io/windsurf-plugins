---
trigger: always_on
description: > [根目录](../CLAUDE.md) > **codeagent-wrapper**
---

# codeagent-wrapper (Go CLI Wrapper)

> [根目录](../CLAUDE.md) > **codeagent-wrapper**

**Last Updated**: 2026-04-10
**Binary Version**: v5.10.0
**Go Version**: 1.21+（`go.mod:1`）

---

## 模块职责

`codeagent-wrapper` 是用 Go 编写的跨平台 CLI 包装器，将 Codex CLI / Gemini CLI / Claude Code 三种 AI 后端统一成一个标准接口。CCG 工作流系统中的 20+ 个斜杠命令通过调用它来执行多模型协作任务——Claude 作为编排层，codeagent-wrapper 负责实际派发、执行、输出解析和会话管理。

---

## 入口与启动

- **主入口**：`main.go:120` — `main()` 调用 `run()` 获取退出码
- **可执行文件**：
  - macOS/Linux：`codeagent-wrapper`
  - Windows：`codeagent-wrapper.exe`

### CLI 调用语法

```bash
# 单任务模式（新会话）
codeagent-wrapper [--backend <codex|gemini|claude>] "任务文本" [工作目录]

# stdin 模式（处理含换行/特殊字符的任务）
codeagent-wrapper [--backend <codex|gemini|claude>] - [工作目录] <<'EOF'
任务文本
EOF

# 会话恢复
codeagent-wrapper resume <session_id> "任务文本" [工作目录]
codeagent-wrapper resume <session_id> - [工作目录]

# 并行模式（从 stdin 读取多任务配置）
codeagent-wrapper --parallel [--backend <name>] [--full-output] < tasks.txt

# 其他
codeagent-wrapper --version
codeagent-wrapper --cleanup
```

### 参数说明

| Flag | 说明 | 默认值 |
|------|------|--------|
| `--backend <name>` | 指定后端：`codex`、`gemini`、`claude` | `codex` |
| `--gemini-model <name>` | Gemini 型号（仅 gemini 后端有效） | 空（后端默认） |
| `--progress` | 向 stderr 输出紧凑进度行 | 关 |
| `--lite` / `-L` | 精简模式：关闭 Web UI，加快响应 | 关 |
| `--parallel` | 并行模式，从 stdin 读取多任务配置 | — |
| `--full-output` | 并行模式输出完整消息（传统模式） | 关（默认摘要） |
| `--skip-permissions` | 跳过权限提示（claude 后端） | 关 |
| `--version` / `-v` | 打印版本 | — |
| `--help` / `-h` | 打印帮助 | — |
| `--cleanup` | 清理过期日志文件 | — |

### 环境变量

| 变量 | 说明 | 默认 |
|------|------|------|
| `CODEX_TIMEOUT` | 超时毫秒数（>10000）或秒数 | 7200s |
| `CODEX_REQUIRE_APPROVAL` | 启用文件操作审批 | false |
| `CODEX_DISABLE_SKIP_GIT_CHECK` | 禁止跳过 Git 仓库检查 | false |
| `CODEAGENT_ASCII_MODE` | 使用 ASCII 状态符（PASS/WARN/FAIL） | false |
| `CODEAGENT_LITE_MODE` | 精简模式 | false |
| `CODEAGENT_POST_MESSAGE_DELAY` | agent_message 后等待秒数（0-60） | 5s |
| `CODEAGENT_MAX_PARALLEL_WORKERS` | 并行 worker 上限（0=不限） | 0 |
| `GEMINI_MODEL` | Gemini 型号（低优先级，CLI 参数覆盖） | 空 |

### 退出码

| 码 | 含义 |
|----|------|
| 0 | 成功 |
| 1 | 通用错误（参数缺失、无输出） |
| 124 | 超时 |
| 127 | 后端命令不在 PATH |
| 130 | 用户中断（Ctrl+C） |
| 其他 | 后端进程退出码透传 |

---

## 对外接口

### 后端抽象层（`backend.go`）

`Backend` 接口（`backend.go:13`）定义三个方法：

```go
type Backend interface {
    Name()     string
    Command()  string
    BuildArgs(cfg *Config, targetArg string) []string
}
```

已注册后端（`config.go:66`）：

| 后端 | 命令 | 参数构建函数 |
|------|------|------|
| `codex` | `codex` | `buildCodexArgs()` |
| `gemini` | `gemini` | `buildGeminiArgs()` |
| `claude` | `claude` | `buildClaudeArgs()` |

### stdin 传递协议

当任务文本包含以下特殊字符时自动切换 stdin 模式（`main.go:22`）：
- 换行 `\n`、反斜杠 `\`、双引号 `"`、单引号 `'`、反引号 `` ` ``、`$`
- 或任务文本长度 > 800 字符

Gemini 平台差异（`executor.go:864`）：
- **macOS/Linux**：`-p "<任务文本>"` 直接传参（execve 保留多行 argv）
- **Windows**：omit `-p`，通过 stdin pipe 传入（cmd.exe 会截断多行参数，Issue #129）

### 并行任务配置格式（`config.go:109`）

```
---TASK---
id: task-a
workdir: /path/to/project
backend: codex
dependencies: task-b,task-c
---CONTENT---
任务内容文本
---TASK---
id: task-b
---CONTENT---
另一个任务内容
```

支持依赖拓扑排序（`executor.go:287`）：同层无依赖任务并行，跨层串行。

### Session 管理

- 后端执行完成后返回 `SESSION_ID`（写入 stdout 末尾 + 提前写入 stderr）
- 恢复模式：`resume <session_id>` 参数
- 并行模式每个任务独立 session，互不干扰

### WebServer SSE 流（`server.go`）

默认模式（非 `--lite`）启动一个本地 HTTP 服务（随机端口），通过 SSE 实时推送后端输出到浏览器页面，方便观察长任务进度。精简模式（`--lite`）跳过此机制。

---

## 源码结构

### 入口层

| 文件 | 职责 |
|------|------|
| `main.go` | CLI 入口、参数路由、模式分发（单任务/并行/cleanup） |
| `config.go` | `Config` / `TaskSpec` / `TaskResult` 结构体定义；`parseArgs()`；并行配置解析 |

### 后端抽象层

| 文件 | 职责 |
|------|------|
| `backend.go` | `Backend` 接口；`CodexBackend` / `GeminiBackend` / `ClaudeBackend` 实现；`buildClaudeArgs()` / `buildGeminiArgs()`；`loadMinimalEnvSettings()` 读取 `~/.claude/settings.json` 注入环境变量 |

### 执行层

| 文件 | 职责 |
|------|------|
| `executor.go` | `runCodexTaskWithContext()`——核心执行引擎；`commandRunner` 接口（可测试抽象）；并发调度 `executeConcurrentWithContext()`；拓扑排序 `topologicalSort()`；进程终止 `terminateCommand()` / `killProcessTree()`；并行输出报告 `generateFinalOutputWithMode()` |
| `utils.go` | `resolveTimeout()`；输出提取辅助函数（coverage / files / tests / key output） |

### 日志层

| 文件 | 职责 |
|------|------|
| `logger.go` | 异步 Logger（buffered channel + 单 worker goroutine）；日志文件创建于 `os.TempDir()`；`cleanupOldLogs()` 清理过期日志 |

### 解析层

| 文件 | 职责 |
|------|------|
| `parser.go` | `parseJSONStreamInternal()`——流式解析 Codex / Claude / Gemini JSON 事件；提取 `agent_message`、`session_id`；支持进度回调 `onProgressCallback`、session 回调 `onSessionStartedCallback` |
| `filter.go` | `filteringWriter`——过滤 stderr 噪声行（`[STARTUP]`、`YOLO mode`、Node.js warning 等） |

### 服务层

| 文件 | 职责 |
|------|------|
| `server.go` | SSE Web Server；`WebServer` 结构体；`StartSession()` / `EndSession()` / `SendContentWithType()`；浏览器实时预览后端输出 |

### 平台适配层

| 文件 | 构建约束 | 职责 |
|------|----------|------|
| `process_check_unix.go` | `unix \|\| darwin \|\| linux` | `isProcessRunning()`（`syscall.Signal(0)`）；`getProcessStartTime()`（解析 `/proc/<pid>/stat`） |
| `process_check_windows.go` | `windows` | `isProcessRunning()`（`kernel32.dll` + `GetExitCodeProcess`）；`getProcessStartTime()`（`GetProcessTimes` WinAPI） |
| `windows_console.go` | `windows` | `hideWindowsConsole()`——`CREATE_NO_WINDOW`（`0x08000000`）防止 taskkill 弹出 CMD 窗口 |
| `windows_console_unix.go` | `!windows` | `hideWindowsConsole()` 空实现 |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fengshao1227/ccg-workflow](https://github.com/fengshao1227/ccg-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
