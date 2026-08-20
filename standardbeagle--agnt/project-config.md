---
trigger: always_on
description: Claude Code 治此 repo 之綱：唯導向與不變式；詳見 `docs/`、`.claude/rules/`（見 [Reference Map](#reference-map)）。
---

# CLAUDE.md

Claude Code 治此 repo 之綱：唯導向與不變式；詳見 `docs/`、`.claude/rules/`（見 [Reference Map](#reference-map)）。

## Project Overview

**agnt**：賦 AI coding agents 以 browser superpowers；通 AI agent 與 browser，作 real-time debug、UI wireframe、visual feedback。

- **Version**: 0.15.4
- **Repository**: https://github.com/standardbeagle/agnt

**Binaries**:
- `agnt`: 主 CLI（唯一實建 binary）
- `agnt-daemon`: daemon auto-start 副本（避 sandbox 禁 fork）
- `devtool-mcp`: 舊名 alias（backwards compat）

**CLI Subcommands**: `mcp` (MCP server), `serve`, `run` (PTY wrapper), `daemon`, `session` (`hosts` lists detachable sessions; `kill` terminates one), `attach` (detach/reattach a daemon-owned PTY), `ssh` (remote session-host client with reconnect and forwarding), `push` (SFTP delivery to an active remote session), `init` (setup-only, no relaunch), `skills` (install agnt skills via `npx skills` + register MCP), `shim` (shim watcher/install), `monitor` (event stream), `errors`, `ci`, `doctor`, `notify`, `up`, `upgrade`, `completion`, `ai` (interactive AI — Claude-only, stream-json), `acp` (any ACP agent via `coder/acp-go-sdk`: gemini/opencode/claude-code-acp; one-shot + overlay/cooked REPL; fs+terminal caps; deterministic alert gate), `hook` (telemetry forwarder), `activate`/`license` (Pro license activation + management — offline lk validation, see `internal/license/`)。`setup-project` is a skill (`/agnt:setup-project`), not a cobra command.

**Core Architecture Decisions**:

1. **Binary copies instead of self-exec**：sandbox（如 Claude Code）禁 binary fork/exec self；別本 binary 可避之。
2. **`agnt run` workaround for MCP notifications**：MCP servers 不能 push notifications；`agnt run` 以 PTY 包 AI tools，注 browser events 為 synthetic stdin：
   ```
   Browser → Proxy → HTTP POST → Overlay (port 19191) → PTY stdin → AI Tool
   ```
3. **System prompt/context delivery**：起 AI agents 時自注或持久化 agnt context；Claude Code 用 `--append-system-prompt`；Gemini, Copilot, Aider 等 normal sessions 寫入 agent context file，setup mode 才用 stdin prompt。詳 `docs/agent-adapters.md`。

**Core Features**: browser debug (screenshots, DOM inspect, error capture), floating indicator messaging, sketch mode (Excalidraw-like wireframe), design mode (AI UI iteration), process/proxy management with daemon persistence, PTY overlay, shell shims (`.agnt/bin` PATH wrappers routing dev/build/kill through the daemon; `internal/shims/`, SHIM verb).

## Installation

```bash
# Install binary
go install github.com/standardbeagle/agnt/cmd/agnt@latest
# or: make install-local

# Register MCP
claude mcp add agnt -s user -- agnt mcp
```

Claude Code plugin 已遷 standalone marketplace repo；此 repo 唯出 `agnt` binary + MCP server。

**MCP Config** (`claude_desktop_config.json`): `"agnt": {"command": "agnt", "args": ["mcp"]}`

**Project Setup**: `/agnt:setup-project` (auto-detects project, configures auto-start)

## Build Commands

```bash
make build          # Build agnt binary
make all            # Build + create binary copies
make test           # All tests (except procisolation-tagged)
make test-isolated  # procisolation tests inside PID namespace
make test-chrome-e2e # real-Chrome tests on an unloaded machine
make test-coverage  # Generate coverage.html
make install-local  # Install to ~/.local/bin

# Single package tests
go test -v ./internal/daemon
go test -race ./...
```

## Architecture

### Five-Layer Design

1. **MCP Tools** (`internal/tools/`): daemon-aware MCP tools
2. **Daemon** (`internal/daemon/`): background service, persistent state, socket IPC
3. **Protocol** (`internal/protocol/`): text IPC protocol
4. **Business Logic** (`internal/project/`, `internal/proxy/`): project detection, reverse proxy. Process management lives in the vendored `github.com/standardbeagle/go-cli-server/process` (ProcessManager, ManagedProcess) — there is no `internal/process` package.
5. **Infrastructure** (`go-cli-server/process` RingBuffer, `internal/config/`): RingBuffer, config

### Critical Design: Lock-Free Process Management

**ProcessManager**：`sync.Map` registry；`atomic.Int64` metrics (`activeCount`, `totalStarted`, `totalFailed`)；`atomic.Bool` shutdown coordination。

**ManagedProcess**：state 皆 atomic：`atomic.Uint32` (`state`), `atomic.Int32` (`PID`/`exitCode`), `atomic.Pointer[time.Time]` (`timestamps`)。唯 RingBuffer boundary writes 用一 `sync.Mutex`。

### Process Lifecycle State Machine

```
Pending → Starting → Running → Stopping → Stopped/Failed
              ↓                     ↓
          Failed ←──────────────────┘
```

State transitions 必由 `CompareAndSwapState()` atomic。Child cleanup：process groups (`Setpgid: true`) + `signalProcessGroup()` 殺 parent + children。

### Reverse Proxy Architecture

**ProxyServer** (`internal/proxy/server.go`)：基 `httputil.ReverseProxy`；HTML response 注 JS；WebSocket server for frontend metrics (`/__devtool_metrics`)；`sync.Map` registry；auto-port discovery；auto-restart max 5/min。

四部：1 HTTP proxy forwards/logs/modifies；2 JS injection（error tracking, `__devtool` API）；3 WebSocket server receives metrics；4 JS execution (`proxy exec` browser control)。


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [standardbeagle/agnt](https://github.com/standardbeagle/agnt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
