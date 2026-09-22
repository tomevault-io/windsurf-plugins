---
trigger: always_on
description: This repository builds `wmux`: a cross-OS terminal multiplexer with a
---

# AGENTS.md

This repository builds `wmux`: a cross-OS terminal multiplexer with a
Windows-first implementation path and an OS-neutral core.

Architecture and behavior are defined by wmux's product requirements, tests,
documented contracts, relevant standards, and official platform documentation.
Prior art may inform research, but another product must not define wmux's
identity or compatibility contract.

The goal is to build a persistent terminal virtualization server:

```text
persistent server
  -> disposable clients
  -> sessions
  -> windows
  -> panes
  -> server-owned virtual terminal state
  -> renderer per attached client
```

## Product Ethos

`wmux` is a serious terminal multiplexer, not a toy pane splitter.

It should let developers keep long-running shells, servers, logs, build jobs,
agents, and project workspaces alive across terminal restarts.

The product promise is:

```text
Consistent multiplexer semantics across supported operating systems.
Native platform integration for terminals, processes, and IPC.
No POSIX emulation dependency.
Core designed for cross-OS support from day one.
```

The wrong promise is byte-for-byte equivalence between Unix and Windows PTYs,
signals, process groups, shell job control, or file descriptor passing.

## Non-Negotiable Architecture Rules

1. Core code must not import Windows APIs.
2. Core code must not import Unix APIs.
3. Core objects use stable IDs, not raw platform handles.
4. The server is the only authority for sessions, windows, panes, grids,
   layouts, options, command queues, paste buffers, jobs, and clients.
5. Clients are disposable views. Losing a client must not kill pane processes
   unless explicitly requested.
6. A pane is a virtual terminal, not just a child process.
7. Child output is parsed into an internal screen/grid. The attached terminal is
   only a renderer.
8. Commands mutate state through a serialized server-side command queue.
9. IPC is versioned from the start.
10. Platform mechanisms are exposed to core as OS-neutral semantic events.
11. Windows first means Windows backend first, not Windows-only architecture.

## Critical Boundary

The platform backend owns OS mechanics only:

```text
ConPTY / PTY handles
process spawning
process cleanup
named pipes / Unix sockets
terminal raw mode
console or termios mode restoration
credential checks
```

The core owns multiplexer semantics:

```text
sessions
windows
winlinks
panes
layouts
screen/grid
VT parser
redraw model
commands
options
formats
key tables
copy mode
paste buffers
hooks
control mode
```

Do not repeat the old design mistake where the PTY/process wrapper owns the
terminal engine or screen. The PTY backend should emit bytes/events. The core
pane should own parser, screen, grid, scrollback, mode state, and dirty state.

Correct flow:

```text
ConPTY bytes
  -> PtyEvent::Output
  -> core pane VT parser
  -> screen-write operations
  -> pane screen/grid
  -> redraw builder
  -> client terminal renderer
```

Incorrect flow:

```text
ConPTY object
  -> owns terminal grid
  -> renderer reaches into platform process for screen state
```

## Rust Workspace Shape

Preserve the crate boundaries:

```text
crates/wmux-core
  OS-neutral server state, panes, sessions, layouts, parser, grid, commands.

crates/wmux-protocol
  Versioned framed IPC messages and codec.

crates/wmux-platform
  OS-neutral platform traits and semantic event types.

crates/wmux-windows
  Windows ConPTY, CreateProcessW, Job Objects, named pipes, console modes.

crates/wmux-unix
  Unix PTY, process groups, signals, Unix sockets, and termios.

crates/wmux-server
  Persistent daemon/server runtime.

crates/wmux-client
  Disposable CLI/attach client.

crates/wmux-cli
  Shared CLI parsing.

crates/wmux-config
  Configuration loading, defaults, and validation.

crates/wmux-bench, crates/wmux-conformance, crates/wmux-stress
  Performance, semantic-conformance, and deterministic stress harnesses.
```

## Server State Model

Use stable IDs and centralized stores.

Prefer:

```rust
struct ServerState {
    sessions: Store<SessionId, Session>,
    windows: Store<WindowId, Window>,
    winlinks: Store<WinlinkId, Winlink>,
    panes: Store<PaneId, Pane>,
    clients: Store<ClientId, Client>,
}
```

Avoid cyclic Rust ownership graphs. Keep ownership in centralized stores and
resolve relationships through stable IDs. Indexes are display metadata only;
they are not ownership identity.

## Event And Command Discipline

The server event loop is the only state mutator.

Allowed event producers:

```text
PTY readers      -> pane output events
IPC handlers     -> client command / attach input events
resize watchers  -> client resize events
timers           -> timer events
terminal readers -> client input events
```

Forbidden:

```text
PTY reader directly changing active pane
IPC handler directly killing panes
renderer directly mutating layout
input decoder directly mutating sessions/windows
random task touching server-owned state
```

All user actions should become commands. Commands must resolve targets through
a target-resolution subsystem, not through ad hoc lookups in each command.

## Pane And Terminal Model

A pane owns a virtual terminal:

```text
pane id
window id
layout cell
backend pane id
terminal parser

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shreshthkapai/wmux](https://github.com/shreshthkapai/wmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
