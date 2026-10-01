---
trigger: always_on
description: This file defines how automated coding assistants should work in this repository. Treat it as a practical operating manual, not a substitute for reading the code.
---

# Working Agreement for hrdx

This file defines how automated coding assistants should work in this repository. Treat it as a practical operating manual, not a substitute for reading the code.

## Product intent

hrdx is an experimental, minimal, lightweight terminal multiplexer for coding agents and shells. Changes should preserve its defining properties:

- one portable Go binary
- every pane backed by a real PTY
- projects represented as workspaces with tabs and split layouts
- terminal behavior compatible with ordinary agent TUIs and shells
- persistent sessions through a lightweight holder process
- predictable behavior on macOS, Linux, and Windows
- a small dependency and operational footprint

Prefer a narrow, explicit implementation over a generalized subsystem. New abstractions must earn their cost through a real ownership boundary or repeated use.

## Starting a task

Build context before editing:

1. Inspect `git status --short --branch`. Existing modifications may belong to the user or another agent.
2. Locate and read every `AGENTS.md` that governs the target path.
3. Read the owning implementation, nearby tests, and user-facing documentation for the behavior.
4. Reproduce reported failures when feasible. Keep expected behavior distinct from observed behavior.
5. For GitHub work, inspect the issue, discussion, or pull request before implementing it. Do not change branches merely to review a pull request.

Do not use one search result or one function as a complete model of a feature. Follow the path through UI state, terminal or holder behavior, persistence, API representations, tests, and platform-specific files where relevant.

## Code ownership map

Put behavior in the package that owns the concern:

| Area | Responsibility |
|---|---|
| `cmd/hrdx` | CLI flags, configuration assembly, startup, process modes, and platform wiring |
| `internal/api` | Socket server, newline-delimited JSON protocol, public request and response types, subscriptions |
| `internal/holder` | Persistent PTY ownership, attach and detach, replay ring, holder protocol |
| `internal/state` | Serializable workspaces, tabs, panes, layouts, and preferences |
| `internal/term` | PTY pane lifecycle, input encoding, resize, scrollback, selection, terminal-facing helpers |
| `internal/ui` | Bubble Tea model, rendering, layout, input, menus, settings, persistence coordination, API handling |
| `internal/vt` | Escape parsing, terminal state, glyphs, colors, history, and screen model |
| `internal/update` | Update checks and self-update |
| `internal/winproc` | Windows foreground-process and process-tree behavior |

The API server must not mutate UI state directly. Terminal escape parsing must not leak into UI layout code. Persisted structures belong in `internal/state`, while conversion between live and saved state belongs in `internal/ui/persist.go`.

## Correctness contracts

### UI state and API ordering

- Bubble Tea's update loop owns live UI state. Keep mutations on that loop.
- Socket requests must round-trip through `api.Request` and a buffered reply channel.
- API and keyboard actions should use the same model operations and preserve the same invariants.
- Event publication must not block rendering. Best-effort subscribers may drop events when slow.
- Modal input modes must consume only their own keys. Terminal mode should continue forwarding ordinary input to the focused PTY.

### Layout and rendering

- Every persistent pane in a tab must appear exactly once in its split tree.
- Removing a split leaf must promote its sibling without corrupting ratios.
- Floating or otherwise ephemeral panes must not enter persisted split trees unless the contract explicitly changes.
- PTYs receive the content dimensions inside pane borders, not the outer rectangle.
- ANSI-styled rows must be clipped and padded by display width. Byte length and rune count are insufficient.
- Account for wide glyphs, combining characters, cursor visibility, narrow windows, and stale PTY sizes.
- Overlays must preserve ANSI reset behavior and mouse hit testing must follow visual stacking order.

### PTYs and holder sessions

- A directly owned PTY dies with its pane or TUI. A holder-backed persistent PTY detaches when the TUI quits and reattaches on restart.
- Explicit pane, tab, or workspace close must clean up the corresponding process and model references.
- Ephemeral panes must not leave detached holder sessions behind.
- Resize, output replay, process exit, attach failure, holder restart, and stale session cleanup are separate paths and should remain testable.
- Avoid goroutine leaks and blocking sends in process and socket paths.

### Terminal emulation and input

- Preserve valid VT state across partial escape sequences and arbitrary feed boundaries.
- Input encoding must respect application cursor mode, bracketed paste, mouse capture, and kitty keyboard behavior.
- Escape and control keys are application input in terminal mode. Do not reserve them globally without an explicit mode or prefix contract.
- Selection, scrollback, alternate-screen behavior, and child mouse capture should match normal terminal expectations.

### Persistence


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [patriceckhart/hrdx](https://github.com/patriceckhart/hrdx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
