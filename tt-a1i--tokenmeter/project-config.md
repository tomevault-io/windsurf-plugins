---
trigger: always_on
description: TokenMeter 是一个 AI 编码 Agent 本地用量仪表盘，监控 Claude Code 和 Codex 的 Token 消耗、费用、工具调用、文件变更等。
---

# TokenMeter — 开发指南

## 项目概述

TokenMeter 是一个 AI 编码 Agent 本地用量仪表盘，监控 Claude Code 和 Codex 的 Token 消耗、费用、工具调用、文件变更等。

**技术栈：** Go 1.24 · Bubbletea TUI · SQLite (modernc, 纯 Go) · Unix socket

---

## 架构

```
Claude Code hooks ──→ tokenmeter emit ──→ Unix socket (~/.tokenmeter/tokenmeter.sock)
                                           │
Codex JSONL 日志 ──→ CodexWatcher ─────────┘
                                           │
                                     tokenmeter daemon
                                           │
                                    SQLite (~/.tokenmeter/data/tokenmeter.db)
                                           │
                                     tokenmeter TUI (bubbletea)
                                           │
                                     tokenmeter web (HTTP Dashboard)
```

### 组件职责

| 组件 | 文件 | 职责 |
|------|------|------|
| `tokenmeter emit` | `cmd/tokenmeter/main.go:runEmit` | Claude hook 的轻量接收端，读 stdin → 发 Unix socket |
| daemon | `internal/daemon/daemon.go` | 接收事件、存 SQLite、广播给订阅者 |
| collector/claude | `internal/collector/claude.go` | Claude hook 事件解析 → 统一 Event |
| collector/codex | `internal/collector/codex.go` | 轮询 Codex JSONL 日志 → 统一 Event |
| collector/cost | `internal/collector/cost.go` / `pricing.go` | Claude / Codex 费用估算 |
| storage | `internal/storage/` | SQLite schema、读写、查询 |
| tui | `internal/tui/*.go` | Bubbletea TUI，4 个 tab |
| web | `internal/web/` | HTTP Dashboard（REST API + 嵌入式 SPA 前端） |
| event | `internal/event/event.go` | 统一事件类型定义 |

---

## 构建与运行

```bash
make build          # 编译到 ./tokenmeter
make install        # 编译并复制到 $GOPATH/bin
go test ./...       # 运行所有测试
go vet ./...        # 静态检查

tokenmeter setup         # 向 ~/.claude/settings.json 注入 hooks
tokenmeter               # 启动 TUI（自动启动 daemon）
tokenmeter daemon        # 仅启动 daemon
```

**本地调试 Claude hooks：**
```bash
echo '{"hook_event_name":"PreToolUse","session_id":"test-123","tool_name":"Read","tool_use_id":"tu-abc"}' | tokenmeter emit
```

---

## 数据库 Schema

5 张表，均在 `internal/storage/db.go:migrate()` 中用 `CREATE TABLE IF NOT EXISTS` 创建：

- `sessions` — 会话（session_id PK, platform, start_time, status, total_input_tokens, total_output_tokens, total_cost_usd）
- `agents` — Agent 实例（agent_id PK, session_id FK, parent_agent_id, role, status）
- `tool_calls` — 工具调用（call_id PK, agent_id, session_id FK, tool_name, params_summary, result_summary, duration_ms, status）
- `token_usage` — Token 记录（id AUTOINCREMENT, agent_id, session_id FK, input_tokens, output_tokens, model, cost_usd）
- `file_changes` — 文件变更（id AUTOINCREMENT, session_id FK, file_path, change_type）

**Schema 变更规则：** `migrate()` 以 `CREATE TABLE IF NOT EXISTS` / `CREATE INDEX IF NOT EXISTS` 为主，并通过 `addColumnIfMissing()` 做幂等 `ALTER TABLE ADD COLUMN`。允许新增带默认值的兼容列；不允许删除或重命名已有列。

---

## Claude Code Hook 集成

Claude Code 向每个 hook 的 stdin 发送 JSON。已注册的 hooks：

```
SessionStart, SessionEnd, Stop,
PreToolUse, PostToolUse, PostToolUseFailure,
SubagentStart, SubagentStop
```

### 关键字段

| Hook | 关键字段 |
|------|---------|
| SessionStart | `session_id`, `cwd` |
| PreToolUse | `session_id`, `agent_id`, `tool_name`, `tool_input` (JSON), `tool_use_id` |
| PostToolUse | `session_id`, `agent_id`, `tool_name`, `tool_result`, `tool_use_id` |
| PostToolUseFailure | 同上 |
| Stop | `session_id`, `agent_id`, `reason`, `agent_transcript_path` |
| SubagentStart | `session_id`, `agent_id`, `agent_type` |
| SubagentStop | `session_id`, `agent_id` |

**注意：** Claude hook 事件本身**不携带 token 数量**。Token 数据需通过 `agent_transcript_path`（Stop hook）或扫描 `~/.claude/projects/*/` 下的 JSONL 文件获取。

### Claude JSONL 日志格式

路径：`~/.claude/projects/<cwd-encoded>/<session-id>.jsonl`

**cwd 编码规则：** 将 cwd 中每个 `/` 替换为 `-`（含首字符）。
例：`/Users/admin/code/tokenmeter` → `-Users-admin-code-tokenmeter`

每行一个 JSON 对象。顶层字段示例：

```json
{
  "type": "assistant",
  "sessionId": "126b5856-...",
  "uuid": "d832a967-...",
  "parentUuid": "deef5b7b-...",
  "cwd": "/Users/admin/code/tokenmeter",
  "gitBranch": "main",
  "message": {
    "model": "claude-sonnet-4-6",
    "usage": {
      "input_tokens": 3,
      "output_tokens": 6,
      "cache_creation_input_tokens": 15104,
      "cache_read_input_tokens": 8909
    }
  }
}
```

关注 `type == "assistant"` 的行，从 `message.usage` 取 token 数量。

**只需统计 token 数量，不做费用估算。** 展示字段：
- `input_tokens` + `cache_creation_input_tokens` + `cache_read_input_tokens` → 输入 token 合计
- `output_tokens` → 输出 token 合计

`gitBranch` 字段也出现在顶层（包括非 assistant 类型的行），可用于 session 的人类可读展示名。

---

## Codex 日志格式

`~/.codex/sessions/YYYY/MM/DD/*.jsonl`，每行：`{"timestamp":"...","type":"session_meta|response_item|event_msg","payload":{...}}`

关键 payload 类型：
- `session_meta` → session 开始
- `response_item` → `function_call`（工具调用开始）/ `function_call_output`（工具调用结束）
- `event_msg` with `type:"token_count"` → token 统计

---

## 统一事件模型

所有数据源最终转换成 `internal/event/Event`：

```go
type Event struct {
    ID        string    // tool_use_id (Claude) 或 call_id (Codex)，用于 Pre/Post 关联
    Type      EventType // EventToolCallStart|End|AgentStart|End|TokenUsage|FileChange|SessionStart|End
    SessionID string
    AgentID   string
    Platform  Platform  // "claude" | "codex"
    Timestamp time.Time
    Data      EventData
}
```

Pre/Post 工具调用通过相同的 `ID`（`tool_use_id`）在 daemon 里关联，计算 duration。

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tt-a1i/tokenmeter](https://github.com/tt-a1i/tokenmeter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
