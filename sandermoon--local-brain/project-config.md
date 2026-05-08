---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Local Brain is a minimalist, local-first project management CLI tool for developers. It manages **Brains** (high-level workspaces) and **Projects** (specific focus areas within a Brain), providing context management for notes, tasks, and code repositories.

**Key Concepts:**
- **Brains**: Top-level workspaces (e.g., "Work", "Personal"). Only one brain is active at a time, symlinked to `~/brain`
- **Projects**: Focus areas within a brain. Each has `notes.md`, `todo.md`, and optional git repo links
- **Dump**: Quick capture inbox (`00_dump.md`) for tasks/notes that get refiled to projects
- **Local-first**: Plain text Markdown files, grep-able, version-controllable

## Build & Test Commands

### Building
```bash
make build              # Build binary to ./brain
make dev-install        # Install to ~/.local/bin
make install            # Install to /usr/local/bin (system-wide)
```

### Testing
```bash
make test               # Fast unit tests only (default)
make test-all           # All tests including integration
make test-unit          # Unit tests only
make test-integration   # Integration tests only
make test-cover         # Generate coverage report (coverage.html)
make test-race          # Run with race detection

# Run single test
go test -v ./pkg/api -run TestGenerateItemID
go test -v ./pkg/config -run TestLoadConfig
```

### Code Quality
```bash
make fmt                # Format code
make vet                # Run go vet
make lint               # Run golangci-lint (if installed)
make check              # Run all checks (fmt, vet, test)
```

### Release
```bash
make snapshot           # Test release build without tags (artifacts in dist/)
make release            # Create production release (requires git tag)
```

## Architecture

### Package Structure

**`cmd/`** - Cobra CLI commands (19 commands)
- Each command in separate file (e.g., `add.go`, `project.go`, `todo.go`)
- All register with `rootCmd` in their `init()` functions
- `root.go` defines base command and version handling

**`pkg/api/`** - Core business logic (JSON API layer)
- `dump.go` - Parse dump file, generate stable IDs for items
- `todo.go` - Parse todo.md files, extract tasks with metadata
- `note.go` - Parse notes.md files, extract note entries
- `project.go` - List projects, extract repo URLs from `.repos` files
- `id.go` - MD5-based ID generation (must match bash version for compatibility)

**`pkg/config/`** - Configuration management
- `config.go` - Brain config (JSON), thread-safe with mutex
- `paths.go` - Path resolution with env var overrides (`BRAIN_ROOT`, `BRAIN_SYMLINK`)
- `symlink.go` - Active brain symlink management

**`pkg/fileutil/`** - File operations
- `atomic.go` - Atomic file writes with locking
- `lock.go` - Directory-based file locking (prevents race conditions)
- `platform.go` - Cross-platform utilities (symlinks, path expansion)

**`pkg/external/`** - External tool integration
- `git.go` - Git operations (clone, pull, status)
- `fzf.go` - Fuzzy finder integration for interactive selection
- `tmux.go` - Tmux session management for dev mode
- `editor.go` - Launch user's editor (vim/nvim)

**`pkg/markdown/`** - Markdown parsing
- `parser.go` - Extract tasks/notes from markdown with metadata (`#captured:`, `#done:`)

**`pkg/testutil/`** - Test utilities
- Isolated test brain creation with `t.TempDir()`
- Fixture files in `fixtures/` directory

### Key Design Patterns

**Thread Safety:**
- `pkg/config/config.go` uses `sync.RWMutex` for concurrent access
- File operations use atomic writes + directory locks (`pkg/fileutil/lock.go`)

**Backward Compatibility:**
- Item IDs use same MD5 algorithm as bash version (see `pkg/api/id.go`)
- Config format preserved: `~/.config/brain/config.json`
- File formats unchanged: Markdown with `#captured:`, `#done:` metadata

**Environment Variable Overrides:**
- `BRAIN_ROOT` - Root directory for brains (default: `~/brains`)
- `BRAIN_SYMLINK` - Active brain symlink location (default: `~/brain`)
- `BRAIN_CONFIG_DIR`, `BRAIN_CONFIG_PATH` - Config overrides (mainly for testing)

**Testing Strategy:**
- Unit tests isolated with `t.TempDir()` and env var overrides
- Integration tests verify multi-component workflows
- No test affects user's actual brain data

### Data Flow Example: Refiling

1. **User runs:** `brain refile <id> project-name`
2. **`cmd/refile.go`** parses CLI args
3. **`pkg/api/dump.go`** reads `00_dump.md`, finds item by ID
4. **`pkg/config/config.go`** resolves project path
5. **`pkg/fileutil/atomic.go`** appends to `todo.md` with file locking
6. **`pkg/fileutil/atomic.go`** removes item from dump atomically

### Version Injection

Version info injected at build time via ldflags:
```go
// main.go
var (
    version = "dev"
    commit  = "unknown"
    date    = "unknown"
)

// cmd/root.go - SetVersion() called from main
```

Build: `go build -ldflags "-X main.version=v1.0.0 -X main.commit=abc123 -X main.date=2026-01-28"`

## Important Notes

### CLI/API Parity

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SanderMoon/local-brain](https://github.com/SanderMoon/local-brain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
