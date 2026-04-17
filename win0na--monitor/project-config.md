---
trigger: always_on
description: > **MANDATORY FIRST ACTION — every interaction, no exceptions:**
---

# CLAUDE.md

> **MANDATORY FIRST ACTION — every interaction, no exceptions:**
> Run `git status --short` before doing anything else. If there are no changes, proceed silently. If there are uncommitted user changes, complete the user's request first, then at the end ask if they'd like you to commit those changes along with any changes you made (suggest an appropriate message). Only commit if the user confirms.

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git Conventions

Commit after every logical change. Format: `topic: short description`. For large changes, use a multi-line message where the first line is the short label and the body describes changes in detail.

## Token Optimization

**CRITICAL — apply to every interaction:**

- Read only the lines you need (use offset/limit); never re-read files already in context
- Use targeted Grep/Glob with specific patterns; avoid broad or exploratory searches
- Prefer Edit over Write for small changes; batch related edits when possible
- Keep all output concise — no filler, no restating file contents, no unnecessary explanations
- When multiple tasks arrive in one message, prioritize the latest/most specific request
- Do not echo back code the user can already see; summarize changes instead

## What This Is

A real-time streaming dashboard that monitors OBS stats, YouTube live viewer counts/chat, and GPU utilisation. It compiles to a single native binary (Windows/Linux/macOS) with no runtime dependencies. Serves a responsive web UI designed for phone use during streams.

## Building & Running

Build targets are defined in `magefile.go` using [Mage](https://magefile.org/) — a cross-platform build tool written in Go.

```bash
mage build      # compile for current platform → dist/stream_monitor[.exe]
mage run        # build and run
mage test       # run all tests
mage vet        # go vet ./...
mage fmt        # gofmt -w .
mage lint       # golangci-lint → staticcheck → go vet (first available)
mage coverage   # go test -cover ./...
mage windows    # cross-compile → dist/stream_monitor-windows-amd64.exe
mage linux      # cross-compile → dist/stream_monitor-linux-amd64
mage darwin     # cross-compile → dist/stream_monitor-darwin-arm64
mage clean      # remove dist/
```

**Install Mage:** `go install github.com/magefile/mage@latest` (requires Go).

## Development Setup

After cloning, install optional tooling for the best development experience:

```bash
go install github.com/magefile/mage@latest                          # build system (required)
go install github.com/golangci/golangci-lint/v2/cmd/golangci-lint@latest  # linter (recommended)
go install github.com/evilmartians/lefthook@latest                  # git hooks (recommended)
lefthook install                                                     # activate pre-commit/pre-push hooks
```

**Pre-commit hooks** (via lefthook): `gofmt` and `go vet` run on staged files before every commit. `golangci-lint` and `go test` run before every push.

**CI pipeline** (GitHub Actions): runs lint, test, vet, and cross-compile builds on every push to `main` and on pull requests. Config: `.github/workflows/ci.yml`.

The server starts on port 8888. No YouTube API key is required — it scrapes public pages. Static files (HTML/CSS/JS) are embedded into the binary via `//go:embed`.

## Project Structure

```
stream_monitor/
├── main.go                          # entry point — wires packages, starts goroutines
├── go.mod
├── magefile.go
├── CLAUDE.md
├── static/                          # embedded frontend (HTML/CSS/JS)
│   ├── index.html
│   ├── css/style.css
│   └── js/app.js
└── internal/                        # application packages (not importable externally)
    ├── state/state.go               # shared state structs (OBSState, YTState, GPUState)
    ├── config/config.go             # JSON config persistence + interactive prompts
    ├── obs/obs.go                   # OBS WebSocket v5 client (RFC 6455)
    ├── youtube/youtube.go           # YouTube scraper + live chat via innertube
    ├── gpu/
    │   ├── gpu.go                   # common polling loop
    │   ├── gpu_windows.go           # HWiNFO shared memory → nvidia-smi fallback
    │   ├── gpu_linux.go             # nvidia-smi → sysfs fallback
    │   └── gpu_darwin.go            # IOKit (ioreg) → powermetrics fallback
    └── server/server.go             # HTTP server, static file routes, /stats endpoint
```

## Architecture

Single Go module with domain logic split into `internal/` packages. `main.go` wires everything together.

**Backend** — each `internal/` package owns one concern. Monitoring goroutines receive state pointers from `main.go` and write through exported `Mu` fields (`sync.RWMutex`). The HTTP server reads via `Snapshot()` methods:

- `internal/obs` — WebSocket v5 connection to OBS via raw TCP sockets (RFC 6455); polls stats every 1s, computes rolling 5s bitrate average; auto-reconnects with 3s backoff
- `internal/youtube` — accepts channel handles (`@handle`), video IDs, or full YouTube URLs; validates input at startup; scrapes `/@handle/live` or monitors a video directly; polls viewer count every 30s; live chat via innertube API scraping

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/win0na) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
