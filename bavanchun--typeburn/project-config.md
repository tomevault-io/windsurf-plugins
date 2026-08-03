---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Typeburn is a Monkeytype-style terminal typing test: Go 1.25 + Bubble Tea v2 / Lip Gloss v2, single binary, no backend, local XDG persistence.

## Commands

```sh
make build       # ldflags-stamped binary → ./bin/typeburn
make run         # go run ./cmd/typeburn (launches TUI)
make test        # go test ./...
make test-race   # go test ./... -race -count=1   ← the CI gate; must be GREEN
make lint        # gofmt -l check (must be empty) + go vet ./... + no-TUI guard
make size-check  # build and enforce binary size cap
make version     # build then print the resolved --version banner

# Single test / package
go test ./internal/metrics/ -run TestCompute -count=1
go test ./internal/version/ -run TestResolve_LdflagsWin -v
```

`go test ./... -race -count=1`, `go vet ./...`, and an empty `gofmt -l .` are exactly what CI enforces — run all three before considering work done. UI tests use `teatest` golden files; pure packages are table-driven with real data (no mocks).

## Architecture

**Strict dependency layering — do not violate.** The *pure-logic* packages are UI-free and must stay that way (no `bubbletea`/`lipgloss` imports):

- Pure logic (no UI deps): `internal/typing` (keystroke state machine), `internal/metrics` (WPM/accuracy/consistency formulas), `internal/words` (embedded wordlist + quote pack), `internal/codetext` (Code-mode loader + normalization: `Load(path)` is the ONLY file/stdin I/O boundary; the exported pure `Normalize(string)` shares the same core for in-app paste — keeps words/typing I/O-free), `internal/storage` (atomic JSON persistence), `internal/version` (build stamp).
- Styling/input boundary (intentionally not reusable-core): `internal/config` binds Bubble Tea key types for its keymap, and `internal/theme` returns Lip Gloss styles/colors by design. These two depend on `charm.land` libs deliberately — they are the seam between pure logic and the TUI, not general-purpose libraries. Do not "fix" this by removing the imports; do not add new UI deps to the pure-logic packages above.
- `internal/ui` depends on the packages above and implements the six screen sub-models (Home, Typing, Result, Settings, History, CodePaste) + reusable components.
- `internal/app` is the root Bubble Tea Elm model that wires everything together.

**Elm message flow.** `app.Model` owns a `Screen` enum and six sub-models. Screen sub-models in `internal/ui` emit *domain* messages — `StartTestMsg`, `ResultMsg`, `AbortMsg`, `NavHistoryMsg`, `NavCodePasteMsg`, `CodePastedMsg` (defined in `internal/ui/messages.go`). `ScreenCodePaste` captures one `tea.PasteMsg`, validates via `codetext.Normalize`, and on success emits `CodePastedMsg`; esc is handled by the existing global Back handler (no cancel message). The root model's `Update` routes them, owns screen transitions, and is the *only* place that persists results (`AppendHistory`) and computes new-best. Sub-models never touch storage directly. To add a screen interaction, emit a message from the sub-model and handle routing/side-effects in `internal/app`.

**Metrics derive entirely from the keystroke log.** `typing.Engine` only records keystrokes (`Apply`/`Backspace`, with optional letter-strict mode blocking cursor progression on wrong keys); nothing computes WPM live. `metrics.Compute(log, startMs, durationMs)` replays the log post-hoc, calculating standard `Accuracy` or log-based `KeystrokeAccuracy` for strict runs. Never add live metric mutation — extend the log/replay path instead. `AFKTrim` (Time mode, >7s trailing idle) runs before compute.

**Theme is a semantic `Role` enum, never hex.** UI code asks for `theme.Style(RoleX)` / `theme.Color(RoleX)`. `NO_COLOR` and the `mono` (attribute-only) theme are first-class and must keep layouts identical (only attributes change). Adding a color means adding a `Role` and mapping it in every theme, not a literal in UI code.

**Storage is defensive.** Atomic temp-file + rename; any corrupt/missing file returns safe defaults and never panics; history is capped at the 200 newest records. Settings load once at startup (`app.NewFromDisk()`); history loads on demand and after each test.

**CLI/versioning is hybrid.** `internal/version` reads ldflags-injected `Version/Commit/Date` (set by Makefile + GoReleaser); when empty (bare `go install`) it falls back to `debug.ReadBuildInfo()`, final fallback `"dev"`. `cmd/typeburn/main.go` is a thin fang/cobra entrypoint. The pure `internal/cli.Decide()` preserves v1 root aliases: `--version` short-circuits to the banner; `--text <file>`/`-` selects Code mode. Root-level unknown args still fall through to the TUI; recognized subcommands parse strictly. `-v` is intentionally unbound (reserved for a future `--verbose`).

## Git Workflow (protected main — enforced)

`main` is protected. **Never commit or push directly to `main`.** This is hard-enforced two ways: GitHub branch protection (PR required, direct push denied, `ci.yml` must pass, linear history) and a local PreToolUse hook that blocks `git commit`/`git push` to main in this repo.

Every change — code, docs, config, release prep — follows:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bavanchun/Typeburn](https://github.com/bavanchun/Typeburn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
