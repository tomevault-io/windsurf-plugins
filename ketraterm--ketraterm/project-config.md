---
trigger: always_on
description: This repository is building a modern, secure terminal pipeline in Kotlin/JVM 21.
---

# Terminal Pipeline Agent Guide

This repository is building a modern, secure terminal pipeline in Kotlin/JVM 21.
It is not chasing literal full xterm parity. The goal is a clean, fast,
professional terminal architecture for contemporary shells and TUIs.

Read this file before making changes. Then read the module-level `AGENTS.md`
for the module you touch.

## Architecture

The project is split into strict layers:

- `ketraterm-protocol`: shared protocol constants and small vocabulary types with
  no dependency on parser, core, host, or input.
- `ketraterm-parser`: byte stream to semantic terminal commands.
- `ketraterm-core`: headless terminal state, grid physics, modes, attributes,
  scrollback, width policy, and storage.
- `ketraterm-host`: adapters that map parser semantic commands to core
  APIs.
- `ketraterm-input`: host-bound input encoding for keyboard, paste, focus, and
  future mouse reports.
- `ketraterm-render-api`: dependency-free primitive render frame, cursor, cell,
  cluster, and attribute vocabulary.
- `ketraterm-render-cache`: renderer-side cache that copies primitive render
  frames for UI consumers.
- `ketraterm-transport-api`: dependency-free connector contract for byte-stream
  transports.
- `ketraterm-session`: runtime synchronization point that connects transport,
  parser, host, core response queues, and input encoding.
- `ketraterm-ui-swing`: reusable Swing terminal UI component, painting,
  selection, input event handling, clipboard/font/settings abstractions, and
  viewport/scrollbar model.
- `ketraterm-app`: standalone Swing desktop application hosting the reusable UI
  component, managing tab states, window framing, and PTY processes.
- `ketraterm-testkit`: reusable public test fakes for connector/session tests.
- `ketraterm-pty`: local PTY process lifecycle exposed as transport connectors.
- `ketraterm-benchmarks`: JMH benchmarks for performance-sensitive terminal
  paths.

Keep these boundaries intact:

- Protocol owns shared terminal vocabulary such as ANSI/DEC mode ids and mouse
  mode enums. It must stay dependency-free.
- Parser parses. It owns UTF-8 decoding, ANSI state machines, CSI/OSC/DCS
  recognition, charset shifts, grapheme segmentation, and semantic dispatch.
- Core mutates and stores. It owns cursor physics, margins, wrapping, tab stops,
  scrollback, pen attributes, width calculation, and mode state.
- Integration maps. It must not parse protocols and must not reach into core
  internals.
- Input encodes. It reads stable input-facing mode state and writes host-bound
  bytes without parsing terminal output or touching grid/cursor internals.
- Render API exposes primitive frame contracts only. It must not depend on UI,
  Swing, PTY, parser, host, or core internals.
- Render cache copies render frame data for consumers. It must not choose host
  fonts, parse terminal bytes, or own Swing painting policy.
- Transport connects. Connectors own host I/O threads, deliver raw bytes in
  stream order, synchronously consume host-bound write ranges, and never parse
  terminal protocols.
- Session serializes. It owns parser/core mutation synchronization, drains core
  response bytes, and serializes UI input plus core responses through one
  outbound write lock.
- Swing UI displays and interacts. It must not import IntelliJ APIs, contain
  PTY-specific code, parse terminal output, or know whether bytes come from PTY,
  SSH, tests, or another transport.
  It may use `ketraterm-input` event vocabulary but must send encoded intent
  through `TerminalSession` rather than writing host bytes directly.
- Swing demo hosts. It may start PTY sessions and create windows, but reusable
  rendering and input behavior still belong in `ketraterm-ui-swing`.
- PTY hosts. It starts local pseudo-terminal processes and exposes them through
  `TerminalConnector`. It must not parse protocols, encode input itself, or
  mutate core internals.

Width calculation belongs in core. The parser may assemble grapheme clusters,
but it must not decide how many grid cells a cluster occupies because width
depends on terminal mode and policy.

## Non-Goals

Do not add these unless the product direction explicitly changes:

- Tektronix 4014 emulation.
- Media Copy / printer passthrough.
- X11-specific font protocols.
- Blind OSC 52 clipboard writes.
- Unbounded or unaudited OSC/DCS query responses.
- "Everything xterm ever accepted" compatibility.

Use `docs/terminal-feature-map.md` as the living source for supported features,
and `docs/terminal-feature-gap-map.md` for missing, intentionally deferred,
and policy-gated features.

## Engineering Rules

- Preserve strong SRP. A feature belongs in exactly one responsible layer.
- When implementing or extending query/response features (such as `DECRQSS` or `XTGETTCAP`), or when creating new terminal features that can be queried, always update the explicit security allowlist of queried settings or capabilities, and reject unauthorized or unsupported queries with standard protocol-defined failure responses.
- Prefer the existing module structure and local helper APIs over new patterns.
- Keep hot paths allocation-free or allocation-minimal: primitive arrays,
  packed integers, generated-table-shaped lookups, and explicit buffers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ketraterm/KetraTerm](https://github.com/ketraterm/KetraTerm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
