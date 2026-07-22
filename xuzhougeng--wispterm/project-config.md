---
trigger: always_on
description: WispTerm is a terminal emulator written in Zig, shipping desktop builds for Windows and macOS, plus an experimental Linux AppImage. It uses [libghostty-vt](https://github.com/ghostty-org/ghostty) (Ghostty's VT parser and terminal state machine) for terminal emulation, with its own rendering pipeline (OpenGL/Metal + FreeType, plus DirectWrite/CoreText/fontconfig for font discovery by platform).
---

# AGENTS.md

## Overview

WispTerm is a terminal emulator written in Zig, shipping desktop builds for Windows and macOS, plus an experimental Linux AppImage. It uses [libghostty-vt](https://github.com/ghostty-org/ghostty) (Ghostty's VT parser and terminal state machine) for terminal emulation, with its own rendering pipeline (OpenGL/Metal + FreeType, plus DirectWrite/CoreText/fontconfig for font discovery by platform).

Windows is the **primary and default development target** (`x86_64-windows-gnu`), and day-to-day development happens on Windows in PowerShell. Platform-specific code lives behind narrow interfaces in `src/platform/` (per-platform implementations plus `_unsupported`/`_posix` stubs) so macOS and Linux can share the terminal core. macOS is an active supported desktop build that is still stabilizing; Linux is experimental. See `ROADMAP.md` for future work and `KNOWN_ISSUES.md` for current platform limitations.

## Architecture

WispTerm is split into a platform-agnostic **core** (terminal state, IO, rendering) and a per-platform **host** (window, event loop, input) that drives the core through a narrow surface API. The host interface is `src/platform/window_backend.zig`; OS facilities go through capability facades in `src/platform/`. The named contract — what the host implements, what services it supplies, and the invariants that keep the seam intact — is documented in [docs/architecture.md](docs/architecture.md). Read it before touching the platform boundary or starting a port.

### UI presentation styles

WispTerm has exactly three application UI styles. Choose one before adding a screen; do not create a fourth ad-hoc overlay shape. The shared geometry contract is `src/renderer/ui_patterns.zig`. It is intentionally pure and is included in the fast test suite.

| Style | Use for | Required structure | Examples |
|---|---|---|---|
| **Command palette** | Find-and-run actions with no persistent form state | Search field, filtered and scrollable action list, visible selection, shortcut/meta column, Escape closes. Keep it transient; do not put multi-step configuration here. | Command Center, Copilot History picker |
| **Form dialog** | Small configuration, profile editing, confirmation, and launch choices | Centered constrained dialog, title plus one-line instruction, aligned rows/fields, explicit primary and cancel/close actions. Must fit inside viewport gutters at every size. | New Session, AI/SSH/MCP forms |
| **Workbench page** | Persistent tab-owned tools with browsing, status, or multi-column content | Header, content region, and a full-width footer/status band. Empty states explain why the page is empty and name the next action. Keyboard hints and non-blocking status belong in the footer/status band, never squeezed into a content column. | Settings, Memory Center, Skill Center, Port Forwarding, Agent History |

This follows Ghostty's separation of its command palette dialog from its persistent window/tab shell: Ghostty's palette has a search entry and a scrollable rich list with shortcut metadata, while tabs live in a stable window container. WispTerm uses its GPU renderer rather than GTK, but should preserve that same separation of transient command execution, form input, and persistent workspace state.

When changing a UI page, verify: full labels are visible or deliberately ellipsized with a way to reveal them; the localized UI language is consistent (proper names/model IDs may remain literal); state is placed next to its owner; and the page requests a repaint through `UiEffect` after input changes.

## Cohesion and coupling

These are the **primary** architectural criteria. File length is a symptom, not the rule.

A file may be large when it owns one coherent domain object and exposes a clear API. Ghostty-style large terminal-core files are acceptable because their responsibilities are narrow and their state ownership is explicit — Ghostty ships `terminal/PageList.zig` (~14.8k lines), `terminal/Terminal.zig` (~13.3k), `config/Config.zig` (~10.9k), and `terminal/Screen.zig` (~10.5k) with no scattered module globals. The smell we guard against is **responsibility entanglement**: UI presentation mixed with business mutation, input dispatch mixed with rendering, global mutable state scattered across facades, and one file becoming an import hub for unrelated features. WispTerm's largest files are smaller than Ghostty's yet harder to change because they carry all of those at once.

The goal is not small files but code that can be **understood, tested, and changed locally**. Large-but-cohesive is fine; large-but-tangled must be split. New features should prefer focused modules over growing existing hub files. New UI state must live in an explicit state struct or a feature-owned module — never as another top-level `g_*` / `threadlocal` field in `AppWindow.zig`, `input.zig`, or `renderer/overlays.zig`.

### Integration layer vs feature domains


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuzhougeng/wispterm](https://github.com/xuzhougeng/wispterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
