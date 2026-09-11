---
trigger: always_on
description: This file provides project instructions for coding agents working in this repository.
---

# AGENTS.md

This file provides project instructions for coding agents working in this repository.

## Project Overview

This repository contains the Rune editor/TUI application plus the integrated Rune Agent codebase.

Key entrypoints:

- `cmd/rune` — the main Rune application
- `cmd/rune-agent` — the Rune Agent extension binary and packages
- `auth` — the account contract shared with the API server
- `internal/` — every non-`main` package (`ide`, `text`, `term`, `workspace`,
  `handler`, `component`, `llm`, `cell`, `debug`, ...)

The repository also contains substantial TUI/editor infrastructure built on `github.com/unstablebuild/rune-go-sdk`, and many UI/component patterns mirror the conventions used in the sibling `blue` repository.

New non-`main` packages go under `internal/`. The supported extension API is the
separate `rune-go-sdk` module.

### Language integrations

Before researching, planning, implementing, or reviewing support for a new
programming language, read
[`cmd/rune/docs/docs/develop/languages.md`](cmd/rune/docs/docs/develop/languages.md)
in full. Treat that guide as the required integration checklist, not optional
background reading. It defines how language extensions, project and tool
management, LSP initialization, Tree-sitter assets, core symbol resolution,
debugging, packaging, and the language-specific Rune-core test suites fit
together.

Do not consider a new language or language feature complete unless it follows
the guide's testing pattern. In particular, add or extend the corresponding
language suite in Rune core for each affected layer, using the real language
server, debugger adapter, native syntax queries, and committed `testdata`
project where the guide requires them.

`auth` is the one exception: the account claims, roles and endpoint paths in it
are a wire contract with the API server, which lives in a different module and
therefore cannot import `internal/`. Both ends must deserialize the same token,
so the package is public to keep them from drifting. Keep it that way — nothing
else here is public API, and `auth` should stay free of editor concerns.

## Common Commands

```bash
# Main builds
make                 # Build the repository binaries
make debug           # Build with race detection where applicable

# Individual binaries
make rune
make rune-agent

# Testing
make test            # Run tests with race detector
make test-e2e        # Also run the e2e suites (requires docker)
make coverage        # Generate coverage report

# Code quality
make lint            # Run golangci-lint
make format          # Run go fmt
make generate        # Regenerate generated files
make license

# Good pre-submit validation
make generate && make lint && make test
```

Run a single test:

```bash
go test -race -run TestName ./path/to/package/
```

## License headers

`make license` only adds headers to files missing one. When you add new
files, force-apply the canonical header so a file that already carries
a license-looking but non-canonical comment is rewritten:

```bash
bluectl license -f LICENSE_HEADER <new files>
```

## Repository Architecture

### Main Rune application

- `cmd/rune/` contains the main editor application.
- `cmd/rune/ide/extension/runner.go` is the key integration point for built-in extensions, including Rune Agent.

### Rune Agent

- `cmd/rune-agent/extension/` — workspace extension registration and event handling
- `cmd/rune-agent/dialogue/dialoguemanager/` — conversation orchestration and persistence
- `cmd/rune-agent/dialogue/dialoguetui/` — TUI chat rendering and interaction
- `cmd/rune-agent/llm/` — provider abstraction and model registries
- `cmd/rune-agent/agent/` — agent loop, tools, skills, prompts, task handling
- `cmd/rune-agent/memory/` — memory retrieval and durable memory compilation support
- `cmd/rune-agent/streamiterator/` — gRPC stream iteration helpers

### Key Rune Agent patterns

- Thread safety via `sync.Map` and `sync.Mutex`
- Context cancellation propagated through streaming operations
- Configuration is exposed through the extension config system
- The agent relies heavily on semantic code navigation tools and structural search

## Goroutines

All spawned goroutines must run their body under
`debug.CapturePanicReport` (from this repository's `internal/debug` package) so
that any panic is captured into a crash report and logged instead of
silently taking down the process. This applies to every goroutine
spawn site, including short-lived helpers, background workers, and
goroutines started from production code paths.

```go
go debug.CapturePanicReport(func() {
    // goroutine body
})
```

Do not wrap goroutine bodies in ad-hoc `recover()` blocks in place of
`debug.CapturePanicReport`; the helper is the single source of truth
for panic capture and crash-report generation.

## How to implement a `tui.Component` or `tui.Handler`

The UI elements in this repository are built on `github.com/unstablebuild/rune-go-sdk`.
That SDK is centered around three abstractions:

1. **Component** — drawable/resizable UI element
2. **Handler** — component that also handles events and manages cursor/selection
3. **Event loop** — polls terminal events, routes them, redraws, and flushes output

### Interface flavors


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unstablebuild/rune](https://github.com/unstablebuild/rune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-11 -->
