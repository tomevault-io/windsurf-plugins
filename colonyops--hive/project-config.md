---
trigger: always_on
description: **Hive** is a CLI/TUI for managing multiple AI agent sessions in isolated git environments. Instead of manually managing worktrees, hive handles cloning, recycling, and spawning terminal sessions with your preferred AI tool.
---

# Agent Instructions

## Project Overview

**Hive** is a CLI/TUI for managing multiple AI agent sessions in isolated git environments. Instead of manually managing worktrees, hive handles cloning, recycling, and spawning terminal sessions with your preferred AI tool.

Key capabilities:

- **Session Management** - Create, recycle, and prune isolated git clones
- **Terminal Integration** - Real-time status monitoring of AI agents in tmux
- **Inter-agent Messaging** - Pub/sub communication between sessions
- **Context Directories** - Shared storage per repository via `.hive` symlinks

## Architecture

### Core Concepts

**Session** - An isolated git environment for an AI agent
  - Isolated git clone in a dedicated directory
  - Lifecycle states: active, recycled, corrupted
  - Unique ID and display name
  - Metadata for terminal integration

**Agent** - The AI tool instance (Claude, Aider, Codex) running within a session
  - Detected via terminal output patterns
  - Status monitoring: Active, Idle, Waiting, Error, Missing
  - Future: Multiple agents per session

**Terminal Integration** - Real-time status monitoring (tmux)
  - Maps hive sessions to tmux sessions
  - Captures terminal output for status detection
  - Shows live preview of agent work

**Messaging** - Inter-agent communication via pub/sub
  - Inbox convention: `agent.<session-id>.inbox`
  - Topics support wildcards for broadcast
  - Persistent message storage

### Code Structure

```
internal/
├── commands/       # CLI command handlers (urfave/cli/v3)
├── core/
│   ├── config/     # Configuration loading, validation, defaults
│   ├── git/        # Git operations (clone, pull, status)
│   └── session/    # Session model and Store interface
├── hive/           # Service layer - orchestrates all operations
├── integration/
│   └── terminal/   # Terminal status monitoring (tmux)
├── store/
│   └── jsonfile/   # JSON file session storage implementation
├── tui/            # Bubble Tea TUI (tree view, modals, keybindings)
├── messaging/      # Pub/sub messaging between agents
├── printer/        # Output formatting utilities
└── styles/         # Shared lipgloss styles
```

### Key Files

| File                                        | Purpose                                             |
| ------------------------------------------- | --------------------------------------------------- |
| `main.go`                                   | CLI entry point, global flags, command registration |
| `internal/hive/service.go`                  | Service layer - coordinates sessions, git, rules    |
| `internal/core/config/config.go`            | Config structs, loading, defaults                   |
| `internal/core/config/validate.go`          | Template data structs, validation                   |
| `internal/tui/model.go`                     | TUI model, update loop, view rendering              |
| `internal/tui/tree_view.go`                 | Session tree with status indicators                 |
| `internal/integration/terminal/detector.go` | AI agent status detection patterns                  |

## Development

### Commands

```bash
mise run start            # Run with global config (supports CLI args)
mise run dev              # Run with dev config (supports CLI args)
mise run dev -- new       # Example: run 'hive new' with dev config
mise run build            # Build with goreleaser
mise run test             # Run tests with go test
mise run lint             # Run golangci-lint
mise run check            # tidy + lint + test (full validation)
mise run coverage         # Generate coverage report
mise container            # Build and launch an ephemeral Docker container with hive pre-installed
```

### Manual Testing

Use `mise container` to manually test hive end-to-end. It builds the current branch and drops you into an isolated Docker container with hive installed and tmux available — no need to install a local binary or worry about polluting your dev environment.

```bash
mise container
# Inside the container (hive is aliased to 'hv'):
hv new --remote <url> "my-session"
hv ls
```

This is the preferred way to test CLI/TUI behavior, session creation, branch templates, tmux integration, and anything that requires a real git environment. Do NOT attempt to test by manually building and replacing a binary in your PATH.

### Environment

Dev environment uses `config.dev.yaml` and `.data/` for isolation:

```bash
HIVE_LOG_LEVEL=debug
HIVE_LOG_FILE=./dev.log
HIVE_CONFIG=./config.dev.yaml
HIVE_DATA_DIR=./.data
```

## Code Generation

### go-enum

Enum types use `// ENUM(...)` comments processed by go-enum. Generated files (`*_enum.go`) are committed and must never be edited manually.

```bash
mise run generate:enums    # regenerate after changing ENUM comments
mise run generate          # all generators (go-enum + sqlc)
```

**Defining an enum:**
```go
// ENUM(epic, task)
type ItemType string
```

This generates constants (`ItemTypeEpic`, `ItemTypeTask`), `ParseItemType`, `IsValid`, `MarshalText`/`UnmarshalText`, and `ItemTypeValues`. String values match the ENUM comment exactly (lowercase).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [colonyops/hive](https://github.com/colonyops/hive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
