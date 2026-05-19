---
trigger: always_on
description: This file provides guidance to coding agents collaborating on this repository.
---

# AGENTS.md

This file provides guidance to coding agents collaborating on this repository.

## Mission

Gogg is a command-line and GUI tool for downloading game files from GOG.com.
It authenticates against the GOG API, manages a local catalogue of owned games, and handles file downloads with
progress tracking and resumption support.
Priorities, in order:

1. Correctness of authentication, API interaction, and file downloading.
2. Reliable local state management (SQLite catalogue and token storage).
3. Clean separation between the CLI (`cmd/`), API client (`client/`), persistence (`db/`), and GUI (`gui/`) layers.
4. Cross-platform support (Linux, macOS, and Windows).

## Core Rules

- Use English for code, comments, docs, and tests.
- Prefer small, focused changes over large refactoring.
- Add comments only when they clarify non-obvious behavior.
- Do not add features, error handling, or abstractions beyond what is needed for the current task.
- Keep external dependencies minimal: do not add new `go.mod` entries without prior discussion.

## Writing Style

- Use Oxford commas in inline lists: "a, b, and c" not "a, b, c".
- Do not use em dashes. Restructure the sentence, or use a colon or semicolon instead.
- Avoid colorful adjectives and adverbs. Write "rate limiter" not "smart rate limiter".
- Use noun phrases for checklist items, not imperative verbs. Write "rate limit enforcement" not "enforce rate limits".
- Headings in Markdown files must be in title case: "Build from Source" not "Build from source". Minor words (a, an, the, and, but, or, for, in, on,
  at, to, by, of, is, are, was, were, be) stay lowercase unless they are the first word.

## Repository Layout

- `main.go`: Entry point; initializes the database and delegates to `cmd.Execute`.
- `cmd/`: Cobra command definitions (`cli.go`, `download.go`, `catalogue.go`, `login.go`, `version.go`, `file.go`,
  `gui.go`). Each command wires flags and calls into `client/` or `db/`.
- `client/`: GOG API client — `login.go` (OAuth via chromedp), `games.go` (owned-game listing),
  `catalogue.go` (sync), `download.go` (file downloads with progress), `data.go` (data parsing),
  `rate_limiter.go` (request throttling).
- `auth/`: Authentication service and interfaces wrapping GOG OAuth token lifecycle.
- `db/`: GORM/SQLite persistence — `db.go` (connection setup), `game.go` (game model), `token.go` (token model),
  `repository.go` (data access layer).
- `gui/`: Fyne desktop GUI — `window.go` (main window), `widgets.go` (custom widgets), `theme.go`, `sound.go`,
  `shared.go`.
- `scripts/`: Shell scripts for integration testing and Docker entrypoint.
- `.github/workflows/`: CI workflows for tests and releases.
- `Makefile`: All developer tasks (build, test, lint, format, release).

## Architecture

### Layers

Gogg is organized into four layers that should not have upward dependencies:

1. `db/` — persistence only; no knowledge of the API or CLI.
2. `client/` — GOG API calls and file I/O; depends on `auth/` and `db/` but not on `cmd/` or `gui/`.
3. `cmd/` — Cobra command handlers; orchestrates `client/` and `db/` calls, formats output.
4. `gui/` — Fyne desktop interface; calls into `client/` and `db/` the same way `cmd/` does.

### Authentication Flow

Login is handled by `client/login.go` using chromedp to drive a headless browser through GOG's OAuth flow.
Tokens are stored via `db/token.go` and retrieved by `auth/services.go` for subsequent API calls.

### Download Pipeline

`client/download.go` fetches file metadata, checks for existing partial downloads, and streams bytes with a
`progressbar` wrapper. Resumption is done via HTTP range requests.

### Build Tags

The GUI is gated behind the `desktop` and `gl` build tags. The `headless` tag produces a CLI-only binary with no
Fyne dependency. Use `-tags headless` when running in environments without a display server.

## Go Conventions

- Go version: 1.24 (as declared in `go.mod`).
- Formatting is enforced by `gofmt` (via `make format`) and optionally `gofumpt` (via `make gofumpt`). Run
  `make format` before committing.
- Naming follows Go standard conventions: `PascalCase` for exported identifiers, `camelCase` for unexported
  identifiers and local variables, and `SCREAMING_SNAKE_CASE` for top-level constants where idiomatic.
- Error values are wrapped with context using `fmt.Errorf("…: %w", err)` so callers can use `errors.Is`/`errors.As`.
- Use `zerolog` (already imported as `github.com/rs/zerolog/log`) for all logging; do not use `fmt.Print*` for
  diagnostic output.

## Required Validation

Run the relevant targets for any change:

| Target            | Command                 | What It Runs                                                      |
|-------------------|-------------------------|-------------------------------------------------------------------|
| Format            | `make format`           | `go fmt ./...`                                                    |
| Unit tests        | `make test`             | `go test ./...` with coverage and race detector                   |
| Integration tests | `make test-integration` | `go test -tags=integration ./...`                                 |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [habedi/gogg](https://github.com/habedi/gogg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
