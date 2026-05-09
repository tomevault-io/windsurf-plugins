---
trigger: always_on
description: AgentCommons is a communication layer for orchestrating agent activity across sessions, terminals, providers, accounts, and machines. An attention multiplexer. R1 is the first release: two Claude Code terminals on one machine that can see each other, relay approvals without switching, and maintain persistent identity across session restarts.
---

# AgentCommons -- Build Instructions

AgentCommons is a communication layer for orchestrating agent activity across sessions, terminals, providers, accounts, and machines. An attention multiplexer. R1 is the first release: two Claude Code terminals on one machine that can see each other, relay approvals without switching, and maintain persistent identity across session restarts.

## What R1 delivers

- `/status` in Terminal 1 shows both terminals, their state, and cwd
- When Terminal 2 blocks on an approval prompt, Terminal 1 gets a notification
- `/approve @ZohoMetrics` in Terminal 1 injects the keystroke into Terminal 2 via pty
- `/push @ZohoMetrics "message"` sends a message to Terminal 2 (direct push by default, `--mailbox` alternative)
- `/inbox` lists pending mailbox messages; `/read @name` imports a message into context
- Dual notification modes: direct push (default, real-time) and mailbox (zero context cost, pull on demand)
- Close Terminal 2, reopen it later, slot reconnects automatically, pending messages deliver
- Machine stays awake via caffeinate while agents run
- Daemon auto-launches on first connection (no manual start)
- Terminal names auto-assigned (from Cursor, from context, or from a wordlist)

## Tech stack

- **Go** -- daemon, session wrapper, CLI. Single binary: `commons`. Uses `cobra` for CLI, `creack/pty` for pty, `gorilla/websocket` for WebSocket, `modernc.org/sqlite` for SQLite (pure Go, no CGO).
- **TypeScript** -- MCP server spawned by Claude Code via stdio. Uses `@modelcontextprotocol/sdk` and `ws`.
- **SQLite + WAL** -- `~/.commons/commons.db`. WAL mode for concurrent reads.
- **WebSocket** -- JSON over WebSocket on `localhost:7390` between daemon, MCP servers, and wrappers.

## Project structure

```
├── CLAUDE.md                          # This file
├── go.mod
├── go.sum
├── main.go                            # CLI entry point (cobra root command)
├── cmd/
│   ├── server.go                      # commons server start/stop/status
│   ├── run.go                         # commons run claude (wrapper entry)
│   ├── status.go                      # commons status (query daemon)
│   ├── approve.go                     # commons approve @name
│   ├── deny.go                        # commons deny @name
│   ├── install.go                     # commons install (one-time setup)
│   └── mcp_server.go                  # commons mcp-server (execs node)
├── internal/
│   ├── daemon/
│   │   ├── daemon.go                  # Main daemon process lifecycle
│   │   ├── server.go                  # WebSocket server + /health endpoint
│   │   ├── registry.go                # Slot creation/claiming + session registration
│   │   ├── heartbeat.go               # Heartbeat processing + reaper goroutine
│   │   ├── approval.go                # Approval request broadcast + response routing
│   │   ├── messages.go                # Message storage + slot-addressed queuing
│   │   └── bootstrap.go               # Bootstrap payload generation on slot reclaim
│   ├── db/
│   │   ├── db.go                      # SQLite init, WAL config, migrations
│   │   ├── schema.go                  # CREATE TABLE statements as Go constants
│   │   └── queries.go                 # Prepared query functions
│   ├── wrapper/
│   │   ├── wrapper.go                 # commons run claude orchestration
│   │   ├── pty.go                     # Pty allocation, output/input proxy loops
│   │   ├── detector.go                # Approval pattern regex scanner
│   │   ├── injector.go                # Approval/denial keystroke injection
│   │   └── caffeinate.go              # macOS caffeinate / Linux systemd-inhibit
│   ├── naming/
│   │   ├── naming.go                  # Terminal name resolution (3-layer)
│   │   ├── cursor.go                  # Cursor terminal title detection
│   │   ├── context.go                 # Context-based name generation
│   │   └── wordlist.go                # Memorable wordlist fallback
│   ├── protocol/
│   │   ├── messages.go                # All message type structs (JSON-serializable)
│   │   └── events.go                  # WebSocket event type constants
│   └── config/
│       └── config.go                  # config.toml parsing + defaults
├── mcp/                               # TypeScript MCP server
│   ├── package.json
│   ├── tsconfig.json
│   ├── src/
│   │   ├── index.ts                   # MCP server entry point (stdio transport)
│   │   ├── tools.ts                   # MCP tool definitions (status, push, inbox, read, report_state)
│   │   ├── daemon-client.ts           # WebSocket client + auto-launch + reconnection
│   │   ├── bootstrap.ts               # Bootstrap payload processing on slot reclaim
│   │   └── channels.ts               # Channel protocol for direct push notifications
│   └── approval-patterns.yaml        # Regex patterns for Claude Code approval prompts
└── docs/
    └── approval-patterns.md           # How to add patterns for new CLI tools
```

## Build commands

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DrishtantKaushal/AgentCommons](https://github.com/DrishtantKaushal/AgentCommons) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
