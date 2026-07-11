---
trigger: always_on
description: **Claude Token Monitor** is a pure statusline plugin for Claude Code that displays real-time session information directly in the IDE's status bar. It provides live token usage statistics, git status, tool calls, agent information, and more.
---

# Claude Token Monitor - Project Documentation for Claude Code

## Project Overview

**Claude Token Monitor** is a pure statusline plugin for Claude Code that displays real-time session information directly in the IDE's status bar. It provides live token usage statistics, git status, tool calls, agent information, and more.

### Key Features

- **Token Usage**: Real-time token display with colored progress bar
- **Git Integration**: Branch name and file change statistics (+new ~modified -deleted)
- **Tool Tracking**: Displays active and completed tool calls
- **Agent Info**: Shows active agents and their descriptions
- **TODO Progress**: Tracks completion of TODO items from session
- **Session Duration**: Shows elapsed time for current session
- **Cross-platform**: Supports Windows, macOS, and Linux
- **High Performance**: Stateless execution with <10ms startup time

### Architecture

```
cmd/statusline/            # Statusline plugin entry point
└── main.go               # Entry point with JSON input/output

internal/
├── parser/               # Transcript parsing
│   ├── transcript.go     # JSONL transcript parser
│   └── transcript_test.go
├── statusline/
│   ├── config/           # Configuration management
│   ├── content/          # Content collectors and composers
│   │   ├── folder.go         # StatusLineInput (CC 2.1.150 stdin schema)
│   │   ├── mode_flags.go     # 💭 / ⚡ / effort chip collector
│   │   ├── time.go           # CurrentTimeCollector + tz only (~75 lines)
│   │   ├── version.go        # stdin version fast path + claude --version fallback
│   │   ├── provider.go       # providerKind + detectProvider + cache match
│   │   ├── quota.go          # QuotaCollector + UsageData/MCP types + render
│   │   ├── quota_cache.go    # file-backed cache + 429 backoff state machine
│   │   ├── quota_http.go     # proxy + HTTP client + parseRetryAfterHeader
│   │   ├── quota_anthropic.go # Anthropic OAuth fetcher + stdin rate_limits fast path
│   │   ├── quota_glm.go      # GLM monitor fetcher + plan-window metadata
│   │   └── composers/        # Cell composers (token, git, time-quota)
│   ├── layout/           # Layout management
│   └── render/           # Output rendering
└── windows/              # Windows console initialization
```

### CC 2.1.150 Stdin Schema (StatusLineInput)

Beyond the original fields, the plugin now also consumes:

| Field | Type | Used by |
|-------|------|---------|
| `version` | `string` | `version.go` — echoed; skips `claude --version` fork |
| `rate_limits` | `*StdinRateLimits` | `quota_anthropic.go` — Anthropic quota; skips OAuth `/api/oauth/usage` request |
| `effort.level` | `string` (`low` / `medium` / `high` / `xhigh` / `max`) | `mode_flags.go` — colored chip |
| `thinking.enabled` | `bool` | `mode_flags.go` — `💭` chip |
| `fast_mode` | `bool` | `mode_flags.go` — `⚡` chip |
| `workspace.repo`, `session_name` | `string` | informational |

Older Claude Code builds that don't send these fields fall back to the original API / subprocess paths automatically — no client-side version sniffing required.

### Claude Code Data Directory (IMPORTANT)

**All platforms** store Claude Code data in `$HOME/.claude/`:

```
~/.claude/
├── projects/           # Session data for all projects
│   ├── C--Users-...-project1/
│   │   ├── session-id-1.jsonl
│   │   └── session-id-2.jsonl
│   └── C--Users-...-project2/
│       └── session-id-3.jsonl
├── settings.json       # Global settings
├── CLAUDE.md           # Global instructions (if exists)
└── hooks/              # Global hooks
```

**Note**: The project directory names are URL-encoded paths (e.g., `C--Users-...-project`).

## Building

**IMPORTANT**: Binaries are built to the **current directory**, NOT to a `bin/` subdirectory.

```bash
# Build statusline plugin (outputs: statusline.exe on Windows, statusline on macOS/Linux)
go build -o statusline.exe ./cmd/statusline
```

### Platform-Specific Output

| Platform | Statusline Binary |
|----------|-------------------|
| Windows  | `statusline.exe`  |
| macOS    | `statusline`      |
| Linux    | `statusline`      |

### Quick Build Command

```bash
# Build for current platform
go build -o statusline$(go env GOEXE) ./cmd/statusline
```

Note: `go env GOEXE` returns `.exe` on Windows and empty on Unix systems.

## Running Tests

```bash
# Run all tests with coverage
go test ./... -coverprofile=coverage.out

# View detailed coverage report
go tool cover -html=coverage.out

# Run tests with race detection
go test ./... -race
```

## Configuration

### Global Configuration (Recommended)

**Location**: `~/.claude/settings.json`

```json
{
  "statusLine": {
    "type": "command",
    "command": "C:\\\\Users\\\\YourName\\\\claude-token-monitor\\\\statusline.exe",
    "env": {
      "STATUSLINE_SINGLELINE": "1"
    }
  }
}
```

### Project-Level Configuration

**Location**: `.claude/settings.json` (project root)

Overrides global settings for specific projects.

### Environment Variables

| Variable | Values | Description |
|----------|--------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [young1lin/claude-token-monitor](https://github.com/young1lin/claude-token-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
