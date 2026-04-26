---
trigger: always_on
description: > Guidelines for AI coding agents working in this Go codebase.
---

# AGENTS.md — slb

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

## Toolchain: Go & Make

We use **Go** and **Make** in this project.

- **Go version:** 1.24+ (as per `go.mod`)
- **Build:** `make build` or `go build ./...`
- **Test:** `make test` or `go test ./...`
- **Format:** Always run `go fmt ./...` before committing
- **Dependencies:** Managed via `go.mod` and `go.sum`
- **Lint:** `golangci-lint run ./...` (via `make lint`)

### Key Dependencies

| Package | Purpose |
|---------|---------|
| `spf13/cobra` | CLI framework (commands, flags, completions) |
| `spf13/viper` | Configuration loading (TOML, env, flags) |
| `BurntSushi/toml` | TOML configuration file parsing |
| `modernc.org/sqlite` | Pure-Go SQLite for request/session/review persistence |
| `charmbracelet/bubbletea` | Terminal UI framework (dashboard, review screens) |
| `charmbracelet/bubbles` | Reusable TUI components |
| `charmbracelet/lipgloss` | TUI styling and layout |
| `charmbracelet/log` | Structured logging |
| `fsnotify/fsnotify` | Filesystem watching for daemon mode |
| `google/uuid` | UUID generation for request/session IDs |
| `mattn/go-shellwords` | Shell command tokenization and quoting |
| `golang.org/x/term` | Terminal size detection |

### Build Variables

Version, commit, and date are injected via `-ldflags` at build time:

```makefile
LDFLAGS := -ldflags "-X .../cli.version=$(VERSION) -X .../cli.commit=$(COMMIT) -X .../cli.date=$(DATE)"
```

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

New files are reserved for **genuinely new functionality** that makes zero sense to include in any existing file. The bar for creating new files is **incredibly high**.

---

## Backwards Compatibility

We do not care about backwards compatibility—we're in early development with no users. We want to do things the **RIGHT** way with **NO TECH DEBT**.

- Never create "compatibility shims"
- Never create wrapper functions for deprecated APIs
- Just fix the code directly

---

## Compiler Checks (CRITICAL)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/slb](https://github.com/Dicklesworthstone/slb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
