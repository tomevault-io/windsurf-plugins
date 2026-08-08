---
trigger: always_on
description: Guidance for AI agents and contributors working on **cctop**, an interactive
---

# AGENTS.md

Guidance for AI agents and contributors working on **cctop**, an interactive
`top`-style TUI for monitoring running Claude Code sessions.

## What this is

A single Bun/TypeScript program that lists every running Claude Code session
with process stats, busy/idle state, context size, model, host app, project,
branch, last prompt, a tree of sub-processes, and live sub-agents. On a TTY it's
an interactive TUI; piped or with `--once` it prints one plain-text frame and
exits, while `--json` prints one JSON snapshot and exits.

- **Runtime:** Bun (TypeScript run directly, no build step for dev).
- **Platforms:** macOS and Linux only (process table is read via macOS
  `libproc` FFI or Linux `/proc`).
- **Read-only, with deliberate exceptions.** In its monitoring mode cctop spawns
  no processes and never mutates any session, registry, or transcript. Its only
  writes are its own files under `~/.claude/cctop/`: the usage cache
  (`usage.json`, only under `--capture-usage`) and the persisted TUI preferences
  (`settings.json`, the refresh interval, sort mode, and notifications toggle,
  written only when the user changes them; the file also carries the hand-edited
  `columns` list, which cctop only ever reads — the save merge preserves it); the only thing it ever does to
  another process is send a signal, and only on an explicit user action (`x` →
  SIGTERM a session; `f` → SIGTERM a session's orphaned dev-server processes to
  free their ports). Preserve this property. Separate from all of that is one
  explicit, opt-in mode: the `cctop upgrade` subcommand (`src/upgrade.ts`)
  reaches the network and replaces cctop's own binary. It never runs from the
  refresh loop — the monitor path never even imports it — so everything that
  isn't `cctop upgrade` stays read-only. It is also the only thing that touches
  the network at all: the monitor makes no network calls, a property
  `docs/usage-limits.md` already relies on ("cctop is read-only: it reads
  `~/.claude` and the process table, and makes no network calls") and the reason
  usage limits arrive through a status-line hook instead of an API call. The
  TUI's "restart to run the new version" notice preserves that: it stats its own
  binary (`src/binary.ts`) to see the file was swapped underneath it, rather
  than asking GitHub whether a release exists. Polling from the refresh loop
  would break it.
- **Zero runtime dependencies.** cctop imports only Bun and OS built-ins
  (`bun:ffi`, `node:fs`, …); `package.json` has no `dependencies` field (the
  devDependencies are just Biome/tsc/types). Do not add npm packages — keep it
  dependency-free.

## Bun docs MCP server

This repo ships a project-scoped MCP server in `.mcp.json`.
Use the `bun-docs` MCP to look up Bun APIs and behavior instead of guessing.
It exposes `search_bun` (semantic search) and
`query_docs_filesystem_bun` (`rg`/`cat`/`head` over the docs).

## Commands

Use the Makefile — each target runs the `package.json` script of the same name,
so `make <x>` and `bun run <x>` are interchangeable. (The one exception is
`make prep-release`, Make-only release tooling. `install-bin` is named to match
its script: a script plainly named `install` would fire on `bun install`.)

```sh
make start      # run the TUI (make start ARGS="flux" to pass a filter)
make dev        # run with --watch live reload
make lint       # bun biome check --write . && bun tsc --noEmit  (format + lint + types)
make test       # bun test (the unit suite under test/)
make build      # compile a standalone binary into bin/
make clean      # remove bin/ and stray .bun-build files
make install-bin # compile + install onto PATH (override PREFIX=...)
```

**Always run `make lint` before finishing a change** (it formats, lints, and
type-checks; it exits non-zero on any unfixable issue) — and **`make test`** when
you touch the collectors or renderers. `make lint` does *not* run the tests.

## Layout

```
cctop.ts        entry: CLI arg parsing, the `upgrade` subcommand dispatch, the
                non-interactive paths (--once/--json/-h/-v), dispatch to runApp();
                VERSION derived from package.json
src/app.ts      interactive runtime: runApp(). State, raw-mode input loop,
                draw(), windowGroups(), the quit action
src/render.ts   pure renderers over rows: buildFrame() (summary/header/groups),
                renderDetail(), rowKey(); the column table definition lives here
src/history.ts  pure renderer for the history dashboard (the `h` view): a
                per-day activity bar chart + compact composition text;
                renderHistory() over the aggregated History
src/format.ts   formatting + ANSI helpers (visLen/pad/colors/formatMem/...)
src/notify.ts   "needs you" notifications: pure busy→idle transition tracking
                (finishedSessions) + the BEL/OSC 9 sequence (notifySeq) the
                TUI writes when a session flips to waiting for input
src/upgrade.ts  `cctop upgrade`: the self-updater — resolves the latest release,
                verifies its checksum, and atomically swaps the standalone binary
                (the one place cctop hits the network / rewrites its own binary)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stefanprodan/cctop](https://github.com/stefanprodan/cctop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
