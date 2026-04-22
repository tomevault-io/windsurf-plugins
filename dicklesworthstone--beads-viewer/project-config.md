---
trigger: always_on
description: > Guidelines for AI coding agents working in this Go codebase.
---

# AGENTS.md — beads_viewer

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

## Git Branch: ONLY Use `main`, NEVER `master`

**The default branch is `main`. The `master` branch exists only for legacy URL compatibility.**

- **All work happens on `main`** — commits, PRs, feature branches all merge to `main`
- **Never reference `master` in code or docs** — if you see `master` anywhere, it's a bug that needs fixing
- **The `master` branch must stay synchronized with `main`** — after pushing to `main`, also push to `master`:
  ```bash
  git push origin main:master
  ```

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main:master`

---

## Toolchain: Go & Go Modules

We only use **Go Modules** in this project, NEVER any other package manager.

- **Version:** Go 1.25+ (check `go.mod` for exact version)
- **Toolchain:** `go1.25.5` (see `go.mod`)
- **Dependency versions:** Managed via `go.mod` / `go.sum`
- **Lockfile:** `go.sum` (auto-managed by `go mod`)

### Key Commands

```bash
go build ./...                    # Build all packages
go test ./...                     # Run all tests
go test ./... -race               # Run with race detector
go test ./pkg/analysis/... -v     # Verbose tests for specific package
go vet ./...                      # Static analysis
gofmt -w .                        # Format all Go files
go mod tidy                       # Clean up unused deps
```

### Key Dependencies

| Module | Purpose |
|--------|---------|
| `charmbracelet/bubbletea` | TUI framework (Elm architecture) |
| `charmbracelet/lipgloss` | Terminal styling |
| `charmbracelet/bubbles` | Reusable TUI components (viewport, list, etc.) |
| `charmbracelet/huh` | Interactive form components |
| `charmbracelet/glamour` | Markdown rendering |
| `modernc.org/sqlite` | Pure-Go SQLite for FTS5 search index |
| `gonum.org/v1/gonum` | Graph algorithms (PageRank, betweenness, HITS, eigenvector) |
| `goccy/go-json` | High-performance JSON serialization |
| `fsnotify/fsnotify` | Filesystem event watching (daemon mode) |
| `golang.org/x/sync` | Extended concurrency primitives |
| `pgregory.net/rapid` | Property-based testing |
| `gopkg.in/yaml.v3` | YAML configuration parsing |
| `ajstarks/svgo` | SVG generation for graph export |
| `atotto/clipboard` | Clipboard integration |

### Module Management

- Never manually edit `go.sum`
- Use `go mod tidy` to clean up unused deps
- Dependencies are tracked in `go.mod` / `go.sum`

---

## Code Editing Discipline

### No Script-Based Changes

**NEVER** run a script that processes/changes code files in this repo. Brittle regex-based transformations create far more problems than they solve.

- **Always make code changes manually**, even when there are many instances
- For many simple changes: use parallel subagents
- For subtle/complex changes: do them methodically yourself

### No File Proliferation

If you want to change something or add a feature, **revise existing code files in place**.

**NEVER** create variations like:
- `mainV2.go`
- `main_improved.go`
- `main_enhanced.go`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/beads_viewer](https://github.com/Dicklesworthstone/beads_viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
