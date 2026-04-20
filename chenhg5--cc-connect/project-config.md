---
trigger: always_on
description: CC-Connect is a bridge that connects AI coding agents (Claude Code, Codex, Gemini CLI, Cursor, etc.) with messaging platforms (Feishu/Lark, Telegram, Discord, Slack, DingTalk, WeChat Work, QQ, LINE). Users interact with their coding agent through their preferred messaging app.
---

# CC-Connect Development Guide

## Project Overview

CC-Connect is a bridge that connects AI coding agents (Claude Code, Codex, Gemini CLI, Cursor, etc.) with messaging platforms (Feishu/Lark, Telegram, Discord, Slack, DingTalk, WeChat Work, QQ, LINE). Users interact with their coding agent through their preferred messaging app.

## Architecture

```
┌─────────────────────────────────────────────────┐
│                   cmd/cc-connect                │  ← entry point, CLI, daemon
├─────────────────────────────────────────────────┤
│                     config/                     │  ← TOML config parsing
├─────────────────────────────────────────────────┤
│                      core/                      │  ← engine, interfaces, i18n,
│                                                 │     cards, sessions, registry
├──────────────────────┬──────────────────────────┤
│     agent/           │      platform/           │
│  ├── claudecode/     │  ├── feishu/             │
│  ├── codex/          │  ├── telegram/           │
│  ├── cursor/         │  ├── discord/            │
│  ├── gemini/         │  ├── slack/              │
│  ├── iflow/          │  ├── dingtalk/           │
│  ├── opencode/       │  ├── wecom/              │
│  ├── acp/            │  ├── qq/                 │
│  └── qoder/          │  ├── qqbot/              │
│                      │  ├── line/               │
│                      │  └── weibo/              │
├──────────────────────┴──────────────────────────┤
│                     daemon/                     │  ← systemd/launchd service
└─────────────────────────────────────────────────┘
```

### Key Design Principles

**`core/` is the nucleus.** It defines all interfaces (`Platform`, `Agent`, `AgentSession`, etc.) and contains the `Engine` that orchestrates message flow. The core package must **never** import from `agent/` or `platform/`.

**Plugin architecture via registries.** Agents and platforms register themselves through `core.RegisterAgent()` and `core.RegisterPlatform()` in their `init()` functions. The engine creates instances via `core.CreateAgent()` / `core.CreatePlatform()` using string names from config.

**Dependency direction:**
```
cmd/ → config/, core/, agent/*, platform/*
agent/*   → core/   (never other agents or platforms)
platform/* → core/  (never other platforms or agents)
core/     → stdlib only (never agent/ or platform/)
```

### Core Interfaces

- **`Platform`** — messaging platform adapter (Start, Reply, Send, Stop)
- **`Agent`** — AI coding agent adapter (StartSession, ListSessions, Stop)
- **`AgentSession`** — a running bidirectional session (Send, RespondPermission, Events)
- **`Engine`** — the central orchestrator that routes messages between platforms and agents

Optional capability interfaces (implement only when needed):
- `CardSender` — rich card messages
- `InlineButtonSender` — inline keyboard buttons
- `ProviderSwitcher` — multi-model switching
- `DoctorChecker` — agent-specific health checks
- `AgentDoctorInfo` — CLI binary metadata for diagnostics

## Development Rules

### 1. No Hardcoding Platform or Agent Names in Core

The `core/` package must remain agnostic. Never write `if p.Name() == "feishu"` or `CreateAgent("claudecode", ...)` in core. Use interfaces and capability checks instead:

```go
// BAD — hardcodes platform knowledge in core
if p.Name() == "feishu" && supportsCards(p) {

// GOOD — capability-based check
if supportsCards(p) {
```

```go
// BAD — hardcodes agent type
agent, _ := CreateAgent("claudecode", opts)

// GOOD — derives from current agent
agent, _ := CreateAgent(e.agent.Name(), opts)
```

### 2. Prefer Interfaces Over Type Switches

When behavior differs across platforms/agents, define an optional interface in core and let implementations opt in:

```go
// In core/
type AgentDoctorInfo interface {
    CLIBinaryName() string
    CLIDisplayName() string
}

// In agent/claudecode/
func (a *Agent) CLIBinaryName() string  { return "claude" }
func (a *Agent) CLIDisplayName() string { return "Claude" }

// In core/ — query via interface, fallback gracefully
if info, ok := agent.(AgentDoctorInfo); ok {
    bin = info.CLIBinaryName()
}
```

### 3. Configuration Over Code

- Features that may vary per deployment should be configurable in `config.toml`
- Use `map[string]any` options for agent/platform factories to stay flexible
- Add new config fields with sensible defaults so existing configs don't break

### 4. High Cohesion, Low Coupling

- Each `agent/X/` package is self-contained: it handles process lifecycle, output parsing, and session management for agent X
- Each `platform/X/` package is self-contained: it handles API connection, message receiving/sending, and card rendering for platform X
- Cross-cutting concerns (i18n, cards, streaming, rate limiting) live in `core/`

### 5. Error Handling

- Always wrap errors with context: `fmt.Errorf("feishu: reply card: %w", err)`
- Never silently swallow errors; at minimum log them with `slog.Error` / `slog.Warn`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chenhg5/cc-connect](https://github.com/chenhg5/cc-connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
