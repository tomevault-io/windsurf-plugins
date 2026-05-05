---
trigger: always_on
description: <!-- Parent: ../AGENTS.md -->
---

<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-01-30 | Updated: 2026-02-08 -->

# mnemo

## Purpose

**Memory for AI-assisted development** — Indexes AI coding sessions from 12+ tools (Claude Code, OpenCode, Gemini CLI, Cursor, etc.) into a unified, searchable SQLite database with FTS5 full-text search.

**Status**: Active development, Production-ready
**Version**: 1.3.1

## Key Files

| File | Description |
|------|-------------|
| `main.go` | Entry point — CLI initialization |
| `go.mod` | Go module definition (Go 1.23+) |
| `README.md` | Project overview, install, usage |
| `CONTRIBUTING.md` | Development setup and contribution guide |
| `CHANGELOG.md` | Version history and releases |
| `LICENSE` | MIT License (17588691 CANADA INC.) |

## Project Structure

```
mnemo/
├── main.go
├── cmd/                         # CLI commands (cobra)
│   ├── root.go                  # Root command + configure
│   ├── index.go                 # Indexing orchestrator + onboarding
│   ├── index_helpers.go         # Shared helpers (truncate, inferProvider, etc.)
│   ├── index_claude.go          # Claude Code adapter (JSONL)
│   ├── index_opencode.go        # OpenCode adapter (JSON)
│   ├── index_gemini.go          # Gemini CLI adapter (JSON)
│   ├── index_cursor.go          # Cursor adapter (SQLite)
│   ├── index_codex.go           # Codex CLI adapter (JSONL)
│   ├── index_amp.go             # Amp adapter (JSON + usage ledger)
│   ├── index_crush.go           # Crush adapter (SQLite)
│   ├── index_cline.go           # Cline/Roo/Kilo Code adapter (JSON)
│   ├── index_kiro.go            # Kiro adapter (JSON)
│   ├── index_antigravity.go     # Antigravity adapter (JSONL)
│   ├── index_vscode.go          # VS Code AI chat adapter (SQLite)
│   ├── search.go                # Full-text search command
│   ├── serve.go                 # MCP server (4 tools: search, context, recent, tools)
│   ├── blocks.go                # 5-hour usage block display
│   ├── projects.go              # Project management
│   ├── tools.go                 # Tool detection + path helpers
│   ├── add.go                   # Custom path indexing
│   ├── install.go               # Plugin installer
│   ├── context.go               # Context generation
│   ├── recent.go                # Recent sessions display
│   ├── status.go                # System status display
│   ├── version.go               # Version info
│   └── onboarding.go            # First-run experience
├── internal/
│   ├── db/                      # SQLite database layer
│   │   ├── sqlite.go            # Schema, migrations, init, execer interface
│   │   ├── messages.go          # Message CRUD (with Tx variants)
│   │   ├── sessions.go          # Session CRUD + typed RecentSession queries
│   │   ├── search.go            # FTS5 search + BM25 composite ranking
│   │   ├── projects.go          # Project discovery + classification
│   │   ├── token_usage.go       # Token/cost tracking + typed stats structs
│   │   └── blocks.go            # 5-hour session blocks + usage stats
│   └── tui/                     # Bubble Tea TUI components
│       ├── styles.go            # Catppuccin color palette + shared styles
│       └── projects.go          # Interactive project selector
├── proxy/                       # HTTP proxy for Claude API context injection
│   └── server.go                # Intercepts API calls, injects mnemo context
├── docs/                        # Documentation
├── assets/                      # Media assets
└── scripts/                     # Build and automation scripts
```

## For AI Agents

### Working In This Directory

1. **Adding CLI commands**: Create new file in `cmd/` following cobra pattern
2. **Adding a tool adapter**: Create `cmd/index_<tool>.go`, wire into orchestrator in `cmd/index.go`
3. **Database changes**: Modify relevant file in `internal/db/` (schema changes go in `sqlite.go`)
4. **Testing**: Run `go test ./...` before committing
5. **Building**: Run `go build -o /dev/null .` to verify compilation

### Architecture

```
CLI Commands (cmd/)
  ↓
Tool Adapters (cmd/index_*.go)
  ↓  parse JSONL / JSON / SQLite → atomic transactions
internal/db/
  ├── sqlite.go        → Schema + init + execer interface + BeginTx
  ├── messages.go      → Insert/delete messages (DB + Tx variants)
  ├── sessions.go      → Session tracking (DB + Tx variants)
  ├── search.go        → FTS5 full-text search + BM25 ranking
  ├── projects.go      → Project discovery + classification
  ├── token_usage.go   → Token/cost accounting + typed stats
  └── blocks.go        → Usage block analysis
  ↓
SQLite Database (~/.mnemo/mnemo.db)
```

### Key Design Patterns

- **Cobra CLI**: All commands use cobra framework
- **Bubble Tea TUI**: Interactive experiences use charmbracelet/bubbletea
- **SQLite + FTS5**: Single-file database with full-text search and BM25 ranking
- **One adapter per file**: Each tool gets its own `cmd/index_<tool>.go`
- **MCP Integration**: Model Context Protocol server for Claude Desktop/Cursor
- **Pure Go SQLite**: modernc.org/sqlite — no CGO, no system dependencies
- **execer interface**: Abstracts `*sql.DB` and `*sql.Tx` so insert/delete helpers work with both
- **Atomic transactions**: All indexers wrap delete+insert in a transaction via `BeginTx()` to prevent data loss from partial writes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Pilan-AI/mnemo](https://github.com/Pilan-AI/mnemo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
