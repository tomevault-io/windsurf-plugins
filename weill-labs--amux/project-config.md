---
trigger: always_on
description: See [README.md -- How it works](README.md#how-it-works) for the project overview and system model.
---

# CLAUDE.md

## Design Philosophy

See [README.md -- How it works](README.md#how-it-works) for the project overview and system model.

**Client-server architecture.** The server is a background daemon that owns PTYs and layout state. Clients connect over a Unix socket, receive layout snapshots and raw pane output, and render locally. This enables hot-reload: rebuilding the binary auto-restarts the client with new rendering code while preserving running shells.

**Pane metadata is in-memory.** All pane state lives in `mux.PaneMeta` structs on the server. No external database or state files.

**Names over IDs.** Users reference panes by name (`pane-3`) or numeric ID (`3`). `Window.ResolvePane()` handles resolution. Prefix matches are also supported.

## Architecture

### Key Abstractions

**Client-server protocol** -- Clients send `MsgTypeInput`, `MsgTypeResize`, `MsgTypeCommand`. Server sends `MsgTypePaneOutput` (raw PTY bytes per pane), `MsgTypeLayout` (layout tree snapshot), `MsgTypeRender` (legacy pre-rendered ANSI).

**`mux.Window`** -- Owns the layout tree (`LayoutCell`) and active pane. All layout operations (split, close, resize, focus) go through Window methods.

**`mux.LayoutCell`** -- Binary tree of splits. Leaves hold panes. Internal nodes hold a split direction and children. `Walk()` for traversal, `FindPane()` for lookup, `FixOffsets()` after structural changes.

**`Window.ResolvePane(ref)`** -- Accepts pane name (`pane-1`), numeric ID (`1`), or prefix match. All CLI commands route through this.

**`render.RenderFull()`** -- Composites pane content, borders with junction characters, per-pane status lines, and the global session bar into a single ANSI output string.

### Patterns To Follow

**One package per concern.** Layout logic in `mux/`, rendering in `render/`, server protocol in `server/`. Packages depend on interfaces and shared types (`proto/`), not on each other's internals.

**Unit tests for layout/rendering logic.** See `layout_test.go`, `window_test.go`, `emulator_test.go`. Use `fakePaneID()` helper to create minimal panes for testing.

**Integration tests for end-to-end behavior.** The harness in `test/server_harness_test.go` drives amux directly over the Unix socket -- no tmux dependency. Tests run in ~6s total.

**Inject dependencies, do not add package-level `var` for test seams.** When production code needs a swappable dependency (e.g., clipboard command, time function, exec resolver), pass it as a function parameter or struct field -- never as a mutable package-level `var`. Tests pass stubs directly; production call sites pass the real implementation. This keeps tests parallel and eliminates shared mutable state. See PR #388 for the canonical pattern.

**Use the persistent harness when server lifetime matters.** Prefer `newServerHarnessPersistent()` for integration tests that must keep the server alive independent of client detach timing or transient attachment windows. Use the default harness when exit-on-unattached behavior is part of the behavior under test.

**Guard against impossible states.** Focus fallback finds nearest pane when strict overlap matching fails.

**Save/restore cursor state in copy mode motions.** Compound motions (word, paragraph, etc.) call `moveDown()`/`moveUp()` in scanning loops. These helpers mutate `cy`/`oy` on each call, so the caller must save both values before the loop and restore them when returning `ActionNone`. Otherwise the cursor drifts silently on failed motions.

**Colors live in `config/config.go`.** The Catppuccin Mocha palette (`CatppuccinMocha`), letter abbreviations (`CatppuccinLetters`), and named hex constants (`DimColorHex`, `TextColorHex`) are defined once in the config package. Reference these constants instead of hardcoding hex values like `"f5e0dc"` or `"6c7086"`.

## Development

### Build And Test

```bash
make setup                         # activate repo git hooks
make install                       # install amux (client hot-reloads automatically)
go test ./... -timeout 120s        # run all tests
make coverage                      # merged unit + integration coverage (use this, not go test -coverprofile)
```

### Testing Live

See [README.md -- CLI Reference](README.md#cli-reference) for the full command reference. Key commands for testing:

```bash
amux                              # start or reattach
amux capture --format json        # structured JSON for agents
amux capture --format json pane-1 # single pane JSON
```

### Confirm Before Any Destructive Pane or Daemon Action

**Get explicit user approval before** killing panes, closing windows, restarting orca, cancelling tasks, or any action that destroys worker state. No exceptions. These actions lose running agent context, in-progress work, and session history that cannot be recovered.

Destructive actions that require user confirmation:

- **Killing or closing a pane** (`amux kill`, closing a window)
- **Restarting orca** (`orca stop`, `orca start`) — kills worker panes and orphans active tasks
- **Cancelling an orca task** (`orca cancel`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [weill-labs/amux](https://github.com/weill-labs/amux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
