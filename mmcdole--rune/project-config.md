---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test

```bash
go build ./cmd/rune/    # build
go test ./...           # tests
```

## Architecture

Rune is a MUD client: Go is the kernel (I/O, memory, concurrency), Lua is user space (logic, features, presentation). A single Orchestrator goroutine (the Session) owns the Lua state and processes all events sequentially via channels — thread safety through message passing, not mutexes.

**`docs/architecture.md` is the canonical architecture doc** (system overview, push-model UI, picker, directory structure). Read it before structural changes.

### Event Flow

```
User Submission -> UI input chan -> Session -> rune.hooks.call("input", text, {mode}) -> network
Server Line -> net output   -> Session -> rune.hooks.call("output") -> UI print
Server Prompt -> net output -> Session -> rune.hooks.call("prompt") -> UI prompt overlay
Timer fire -> timer events  -> Session -> rune.timer._fire(id)
Key bind -> UI outbound     -> Session -> rune.binds._dispatch(key)
Bar tick (250ms)            -> Session -> rune.bars._render_all(width) -> UI bars
```

### Go/Lua Boundary Conventions

These rules keep the boundary consistent; follow them when adding APIs:

- **Go registers only `rune._*` primitives** (`_send_raw`, `_timer`, `_input`, `_ui`, ...). Every public name (`rune.send`, `rune.input.get`, `rune.ui.bar`, ...) is defined in Lua, even when the wrapper is thin. The Lua core in `lua/core/` IS the public API surface (loaded in numeric order; each file's header comment states its charter). The only non-underscore fields Go sets are `rune.config_dir` and `rune.version` (data, not API; version is single-sourced from the `version` package so TTYPE/MNES cannot drift from `/version`).
- **Registries live in Lua** on the shared factory (`rune.registry.new`, `15_registry.lua`). Hooks, timers, aliases, triggers, binds, bars, and slash commands all get handles, upsert-by-name, groups, priorities, source attribution, and failure quarantine from one implementation. Go dispatches through internal entry points (`rune.hooks.call`, `rune.binds._dispatch`, `rune.bars._render_all`, `rune.timer._fire`). Dispatch loops that keep iterating after a user callback runs iterate `Registry:snapshot()`, so callbacks may add/remove entries mid-dispatch safely.
- **Presentation belongs to Lua** via `rune.style` (`05_style.lua`). Even the local-echo styling (`"> "` prefix) is a Lua handler on the `"echo"` hook. Go colors only its last-resort degraded-path messages, through `text.Red`/`text.Green` - raw escape codes live in exactly one file per language.
- **Key policy**: Go owns atomic bracketed paste, `Ctrl+Enter`/`Ctrl+J` newline insertion, Enter-to-submit, and editing/cancel keys while a UI-internal mode is active (picker or lossless composer). The ordinary one-line view stays unchanged and has no mode chrome. Application actions remain Lua binds; the composer delegates unhandled chords such as `Ctrl+E`. In normal input, bound printable keys fire only when the input is empty; Go's scroll-key handler is a fallback for unbound keys (keeps degraded mode scrollable).
- **Error convention**: Go primitives return `nil, err` for recoverable failures (send while disconnected, missing file, bad pattern); raising is reserved for programmer errors (wrong argument types).

### Script Robustness

- **Watchdog**: every Go→Lua entry runs under `Engine.guard` with a deadline (`Engine.CallTimeout`, default 5s). Runaway scripts are interrupted with an error; the VM stays usable. Nested entries share the outermost deadline. Host calls that legitimately block on the user (e.g. `open_editor` running `$EDITOR`) run under `Engine.pauseWatchdog`, which detaches the deadline and re-arms a fresh one.
- **Handler isolation**: `rune.hooks.call` runs each handler under `pcall`; a throwing handler is reported and skipped, not allowed to abort the chain.
- **Failure quarantine**: `rune.guarded_call(label, data, fn, ...)` tracks consecutive failures on a registry entry and disables it after 3 in a row. Used by hooks, triggers, aliases, timers, binds, bar renderers, and slash commands. Commands are quarantined individually (`rune.command.dispatch`), so a broken user command can never disable the core input hook.
- **Degraded mode**: if `rune.hooks.call` is unavailable (core script failed, or a user script clobbered `rune.hooks`), the client degrades to a plain telnet client instead of crashing - output passes through raw, input goes to the server, and `/quit` + `/reload` still work.
- **Source attribution**: registrations record the registering script's `file:line` (`rune.caller_source`); it appears in error messages and the `/hooks`, `/triggers`, `/aliases`, `/timers`, `/binds`, `/bars` listings.
- **Engine-level failures** route through `Engine.reportError` → the Lua `"error"` event, with a re-entrancy guard that falls back to direct printing.

### Hook Event Semantics


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmcdole/rune](https://github.com/mmcdole/rune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
