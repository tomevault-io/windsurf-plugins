---
trigger: always_on
description: IMPORTANT: Follow these rules at all times.
---

# CLAUDE.md

IMPORTANT: Follow these rules at all times.

## Critical Rules

- NEVER take shortcuts without asking. STOP, ASK, WAIT for approval.
- NEVER force push. Make a new commit to fix mistakes.
- NEVER commit secrets to version control.
- Only do what was asked. No scope creep.
- Understand existing code before changing it.
- If you don't know, say so. Never guess.
- Test before and after every change.
- Surface errors clearly. No silent fallbacks.
- When rules conflict, prefer the safer, more reversible action and ask.

## Project Conventions

### Branch & Workflow

- All changes go through `develop` — never commit directly to `main`.
- Release flow: `pk changelog` (on `develop`) → `pk release` (merges to `main`, pushes, switches back).
- Conventional Commits required. Configured types: `feat`, `fix`, `deprecate`, `revert`, `security`, `refactor`, `perf`, `docs`, `chore`, `test`, `build`, `ci`, `style`, `plan` (hidden).

### Quick Commands

```bash
make build          # Build for current platform -> dist/pk
make test           # Run tests with race detector
make build-all      # Cross-compile for 5 platforms
make install        # Install to GOPATH/bin
make lint           # Run go vet + gofmt drift check
make vuln           # Scan for known vulnerabilities (govulncheck)
pk changelog        # Generate CHANGELOG.md and commit (no tag)
pk changelog --undo # Unwind an unpushed release commit
pk release          # Read Release-Tag trailer, create tag, merge, and push
```

- **Always use `make build`, never `go build ./cmd/pk` directly.** Bare `go build` drops a binary in the working directory; the Makefile routes output to `dist/`.

### Language & Build

- **Go 1.26**, standard library only — no third-party dependencies.
- Binary: `pk` — single entrypoint at `cmd/pk/main.go`.
- Build: `make build` (output to `dist/`).
- Test: `make test` (runs `go test -v -race ./...`).
- Cross-compile: `make build-all` (darwin/linux amd64+arm64, windows amd64).
- Version injected via ldflags (`-X .../version.version=$(VERSION)`).
- All user messages to stderr, stdout reserved for hook protocol JSON.
- `CGO_ENABLED=0` enforced via Makefile — pure-Go static binaries, no implicit glibc dependency on linux.
- CLI flags use `--kebab-case` (e.g., `--dry-run`, `--project-dir`).

### Directory Structure

- `cmd/pk/` — CLI entrypoint, flag parsing, subcommand dispatch.
- `internal/` — all packages: `changelog`, `config`, `git`, `guard`, `hooks`, `preserve`, `protect`, `release`, `setup`, `status`, `teardown`, `update`, `version`.
- `internal/setup/` — organized by concern: `claude.go` for Claude Code-specific wiring (hooks, settings, bootstrap), `managed.go`/`pin.go`/`baseline.go` for universal logic, `setup.go` for orchestration.
- `docs/` — user-facing documentation. `docs/plans/` — preserved plans (immutable after creation).
- `.claude/skills/` — managed skills (conventions, preserve, ship).
- `.claude/rules/plankit/` — managed rules (development-standards, git-discipline, model-behavior, plankit-tooling), installed under a `plankit/` subdirectory so they never collide with a project's own `.claude/rules/` files (Claude Code discovers rules recursively). `plankit-development.md` (maintainer-only, not shipped) stays at `.claude/rules/`.
- `site/` — landing page.

### Design

- **All commands resolve to the git repository root.** A pk command can be invoked from any subdirectory; it walks up to find `.git` and operates there. Commands don't require being at the root, but they effectively run as if they are. Non-git fallback: when no `.git` exists up the tree, the command uses the provided directory as-is.
- **Safe defaults, opt-in for escalation.** Manual over auto, commit over push — the default should always be the safer, more local action.
- **Three command layers, three flag patterns.**
  - **Hook commands** (guard, preserve, protect, pin) — called by Claude Code automatically. Act immediately; no preview needed.
  - **Skill-managed commands** (ship) — `/ship` handles the preview/confirm cycle for `pk changelog` and `pk release`. `--dry-run` exists for the skill to preview before executing. Power users typing `pk changelog` or `pk release` in the terminal bypass the skill and execute directly.
  - **User-only commands** (teardown) — no skill wrapping, destructive. Preview by default, `--confirm` to execute.

### Code Patterns

- **Dependency injection via Config structs.** Every package exports a `Config` struct with injectable deps (`Stdin`, `Stdout`, `Stderr`, `GitExec`, `ReadFile`, etc.) and a `DefaultConfig()` factory wired to real implementations. DI extends to standalone utility functions too: any function that does file I/O accepts injected `readFile`/`writeFile` parameters rather than calling `os.ReadFile`/`os.WriteFile` directly. The call site in `cmd/pk/main.go` passes the real implementations.
- **Tests use Config mocks** — no external test frameworks, no mocking libraries. Tests inject functions that return canned data. Tests use `t.TempDir()` for filesystem tests. Test error paths, not just happy paths: file I/O failures, git operation failures, and config parse errors all need coverage because they protect against silent data corruption.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [markwharton/plankit](https://github.com/markwharton/plankit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
