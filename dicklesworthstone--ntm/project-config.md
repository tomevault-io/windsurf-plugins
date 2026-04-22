---
trigger: always_on
description: Manages named tmux sessions with agent-aware panes, providing a JSON API (`--robot-*` flags) for AI agents to spawn, inspect, coordinate, and control sessions. Includes a TUI dashboard, CASS integration, and multi-agent ensemble orchestration.
---

# AGENTS.md — ntm

> Guidelines for AI coding agents working in this Go codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Git Branch: `main` Only

**This repository uses exactly one branch: `main`.**

- **All work happens on `main`** — commits, PRs, and automation all target `main`
- **Do not create, use, or sync `master`** — if you see branch logic that references `master`, remove it
- **Do not create or keep side branches like `beads-sync`** unless I explicitly direct it for a temporary workflow

**If you see non-`main` branch handling in code or docs:**
1. Update it to `main`
2. Remove any implication that `master` or another long-lived branch should exist

---

## Toolchain: Go

We only use **Go** in this project. This is a pure Go project — never introduce non-Go tooling for building or testing.

- **Version:** Go 1.25+ (as specified in `go.mod`)
- **Lockfiles:** `go.mod` and `go.sum` only
- **Build:** `go build ./cmd/ntm`
- **Format:** `gofmt` or `goimports`
- **Unsafe code:** Not applicable (Go is memory-safe by design)

### Key Dependencies

| Package | Purpose |
|---------|---------|
| `spf13/cobra` | CLI command framework |
| `charmbracelet/bubbletea` | Terminal UI framework |
| `charmbracelet/bubbles` | TUI component library |
| `charmbracelet/lipgloss` | Terminal styling |
| `charmbracelet/glamour` | Markdown rendering |
| `mattn/go-sqlite3` | SQLite database driver |
| `go-chi/chi/v5` | HTTP router |
| `gorilla/websocket` | WebSocket support |
| `fsnotify/fsnotify` | File system event watching |
| `chromedp/chromedp` | Chrome DevTools Protocol |
| `shirou/gopsutil/v4` | System process utilities |
| `sergi/go-diff` | Text diffing |
| `BurntSushi/toml` | TOML configuration parsing |
| `muesli/termenv` | Terminal environment detection |

### Build & Release

```makefile
# Build for current platform
go build -trimpath -ldflags "-s -w" -o ntm ./cmd/ntm

# Cross-compile (darwin/linux, amd64/arm64)
make build-all
```

### Release Checklist

After creating a new release (via `dsr fallback` or manual cross-compile + `gh release create`):

1. **Verify install script works**: `curl -fsSL ".../install.sh" | bash -s -- --version=vX.Y.Z --dir=/tmp/test --no-shell`
2. **Check flywheel setup checksums**: If `install.sh` content changed, update the SHA256 in `/dp/agentic_coding_flywheel_setup/checksums.yaml` under the `ntm:` entry. If `install.sh` was not modified, no update is needed (the checksum pins the installer script, not the release binaries).

### Logging & Console Output

- Use the standard `log` package or `log/slog` for structured logging.
- No random `fmt.Println` in library code; if needed, make them debug-only and clean them up.
- Log structured context: IDs, session names, pane indices, agent types, etc.
- If a logging pattern exists in the codebase, follow it; do not invent a different pattern.

---

## Code Editing Discipline

### No Script-Based Changes

**NEVER** run a script that processes/changes code files in this repo. Brittle regex-based transformations create far more problems than they solve.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/ntm](https://github.com/Dicklesworthstone/ntm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
