---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, Codex, Cursor, and others) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Codex, Cursor, and others) when working with code in this repository.

## Project Overview

ttt is a terminal text editor written in Go, using tcell for terminal rendering. The Go module is `github.com/eugenioenko/ttt`.

## Build & Test Commands

```sh
make build        # builds to bin/ttt
make run          # build + run
make test         # go test ./...
make fmt          # gofmt -w .
make lint         # golangci-lint run
go test ./internal/core/buffer/   # run tests for a single package

# Open a multi-folder workspace
bin/ttt --workspace project.ttt

# Open specific folders or files
bin/ttt ~/projectA ~/projectB file.go
```

## Architecture

[`ARCHITECTURE.md`](ARCHITECTURE.md) is the source of truth for package ownership and the architecture convergence plan. The codebase uses dependency zones rather than a strict linear layer chain: domain, services, presentation kernel, product presentation, application, plugin host, and platform.

Known boundary violations and explicit boundary decisions are documented there. Highlighting is presentation-owned at `internal/highlight`; Chroma lexing, lexer-state detection, caching, and `term.Style` mapping stay together there. tcell events are intentionally used across `term`, `widgets`, `ui`, and narrow application/platform wiring. Do not create cosmetic wrappers merely to satisfy the old layer diagram.

### Packages

Packages are grouped by the dependency zones in [`ARCHITECTURE.md`](ARCHITECTURE.md), which is the source of truth for zone membership and dependency direction.

**Domain** (`internal/core/`): UI-agnostic editor engine. Domain code must not start processes, access terminal state, render widgets, or coordinate application lifecycle.

- **`core/buffer/`**: line-based text storage (`[]string`), rune-level insert/delete, file I/O (load/save).
- **`core/cursor/`**: visual column cursor with goal-column preservation for vertical movement.
- **`core/undo/`**: command-pattern undo/redo via the `EditCommand` interface; `BatchCommand` groups edits into one undo step.
- **`core/selection/`**: selection ranges and text extraction.
- **`core/multicursor/`**: multi-cursor state (add, dedupe, collapse).
- **`core/fold/`**: indentation-based fold ranges and fold state.
- **`core/diff/`**: line diffing, unified diff generation and parsing, and git gutter change kinds.
- **`core/clipboard/`**: clipboard with system, OSC 52, and process-local backends. It starts processes, which the Domain rules forbid; treat it as an existing exception, not a pattern for new domain code.

**Services**: external-process and external-state integration, exposing typed operations and results without owning widgets.

- **`internal/git/`**: git CLI wrapper (status, staging, commit, repo discovery).
- **`internal/github/`**: `gh` CLI wrapper for pull requests.
- **`internal/lsp/`**: language server client (see LSP Integration).
- **`internal/terminal/`**: integrated terminal emulator. Wraps `gitpod-io/xterm-go` for VT parsing and `aymanbagabas/go-pty` for PTY lifecycle.
- **`internal/watcher/`**: fsnotify-based reporting of on-disk changes to open files and watched directories.
- **`internal/workspace/`**: multi-folder workspaces. `Folder` and `Workspace` track project roots, with `IsRepo` git detection, `FolderForFile` lookup (longest-prefix match), and JSON `.ttt` workspace files. Falls back to `cwd` when no folders are given.

**Presentation kernel**: screen cells, styles, width measurement, rendering, layout, and reusable interaction primitives.

- **`internal/term/`**: `Screen` interface. `TcellScreen` is the real implementation; `MockScreen` supports unit-level `Screen` and renderer tests; `SimScreen` implements tcell's screen contract for composed E2E and chaos tests. Also defines `DirectColor` and `CellAttr` for direct RGB rendering (used by the integrated terminal to bypass the style map for 256-color output).
- **`internal/render/`**: diff-based renderer that compares prev/curr cell grids and emits minimal updates.
- **`internal/textwidth/`**: display-width measurement (`Rune`, `String`, `Runes`), the single source of truth for how many terminal columns text occupies. Wraps `clipperhouse/displaywidth` with the same options tcell v3 uses internally, including the `RUNEWIDTH_EASTASIAN` toggle, so layout always matches what tcell draws.
- **`internal/highlight/`**: presentation-owned per-line syntax highlighting via `chroma/v2`. Owns language selection, multi-line region state (block comments, docstrings, template and raw strings, each discovered by probing the lexer), caching, and mapping Chroma token types to `term.Style`. Full-buffer re-lexing is a known performance trap; avoid it.
- **`internal/view/`**: viewport (scrolling, cursor-to-screen mapping) and the segment-based status bar.
- **`internal/widgets/`**: reusable widget primitives backing both the Plugin Widget API and core panels (tree, table, list, input, dialog, dropdown, tabs, stacks, scrollview, markdown, and so on). `surface.go`/`virtual_surface.go` provide the drawing surface abstraction; `focus.go` handles focus traversal.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eugenioenko/ttt](https://github.com/eugenioenko/ttt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
