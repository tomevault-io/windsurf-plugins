---
trigger: always_on
description: TermSurf is a protocol for embedding web browsers inside terminal emulators. Any
---

# TermSurf

TermSurf is a protocol for embedding web browsers inside terminal emulators. Any
terminal, any browser engine, any TUI — connected by a protobuf/Unix socket
protocol. Users type `web localhost:3000` and see their work without ever
leaving the terminal. No alt+tab, no context switch.

[Agent development guide](https://agents.md/).

## Rules

Do exactly what your user says. No more, no less. NEVER assume they want
something they didn't ask for. NEVER change code unless explicitly asked.

## Vision

TermSurf is a protocol, not just an app. It is a network of interchangeable
components — terminals, browser engines, and TUIs — all speaking the same
protobuf/Unix socket protocol (`termsurf.proto`).

### Cross-platform

TermSurf will work on macOS, Linux, and Windows. iOS and Android can be added
later.

### Every major browser engine

Each browser engine runs as a separate "profile server" process, communicating
with the GUI (terminal emulator) via the TermSurf protocol. One process per
profile.

| Engine   | C library              | Rust binary | Status     |
| -------- | ---------------------- | ----------- | ---------- |
| Chromium | `libtermsurf_chromium` | Roamium     | Done       |
| WebKit   | `libtermsurf_webkit`   | Surfari     | Planned    |
| Gecko    | `libtermsurf_gecko`    | Waterwolf   | Researched |
| Ladybird | `libtermsurf_ladybird` | Girlbat     | Researched |

Each engine follows the same pattern: a C shared library wrapping the engine's
embedding API (`ts_*` functions), linked by a Rust binary that handles Unix
socket IPC, protobuf parsing, and process lifecycle. The Rust binary (~400
lines) is almost entirely reusable across engines.

### Multiple GUIs

TermSurf currently ships as a WezTerm fork (`wezboard/`). We will implement
forks of all major terminal emulators:

- **Ghostboard** — Archived. Will return from a fresh Ghostty fork after the
  protocol stabilizes.
- **Wezboard** (wezboard/) — Active GUI. WezTerm fork, Rust. Full protocol
  support, CALayerHost rendering, input forwarding, DevTools, direct TUI↔Browser
  connection (Issues 715–741).
- **Kitty** — Planned.
- **Alacritty** — Planned.
- **iTerm2** — Planned.

Any terminal that implements the TermSurf protocol can host browser overlays. A
GUI is a terminal emulator that listens on a Unix socket, accepts connections
from TUIs and browser engines, and renders browser content as overlays at pixel
coordinates.

### Many TUIs

The first TUI, `web`, provides browser chrome (URL bar, navigation, modes) in
the terminal pane. But TermSurf is really a webview overlay protocol — many TUIs
can embed web browsers with any engine:

- `web` — General-purpose web browser TUI (current)
- Future TUIs could include: documentation viewers, API explorers, email
  clients, dashboard monitors, or any application that benefits from rendering
  web content inside a terminal.

### The protocol is the product

The TermSurf protocol (`termsurf.proto`) is the most important artifact. It
defines 30 message types covering tab lifecycle, navigation, input forwarding,
GPU compositing, state synchronization, and request/reply pairs. The protocol
will be extended to support:

- All common web browser features (bookmarks, history, downloads, etc.)
- Terminal-specific features (keyboard-based navigation, shrink/grow overlay,
  split management)
- New message types as needs arise

Care goes into the protocol first. Individual apps (boards, engines, TUIs) are
implementations of the protocol.

## Architectural Decisions

### Multi-process architecture

TermSurf is multi-process by necessity, not by choice. Each browser engine
process serves exactly one profile (one set of cookies, storage, and cache).
This is a hard constraint imposed by Chromium (one `BrowserContext` per
process), and Gecko and Ladybird have the same limitation. This constraint is
the defining architectural fact of TermSurf — it shaped every generation from
ts2 onward.

The multi-process design has a second benefit: it enables multi-engine support.
Because each browser process is an independent program speaking the TermSurf
protocol, the GUI doesn't care which engine is behind it. A user can have one
pane running Roamium (Chromium), another running Surfari (WebKit), and a third
running Girlbat (Ladybird) — all in the same terminal window, all speaking the
same protobuf messages.

WebKit is the exception — `WKWebsiteDataStore` supports multiple profiles in one
process — but the one-process-per-profile model still works for it, and keeping
the architecture uniform is more valuable than optimizing for one engine.

**Process topology:**

```
┌─────────┐  ┌─────────┐  ┌─────────┐
│  TUI 1  │  │  TUI 2  │  │  TUI N  │    N TUIs (e.g., `web`)
└────┬────┘  └────┬────┘  └────┬────┘
     │            │            │
     └────────────┼────────────┘
                  │  Unix socket
           ┌──────┴──────┐
           │     GUI     │                1 GUI (terminal emulator)
           │ (Wezboard)  │
           └──┬───┬───┬──┘
              │   │   │
              │   │   │  Unix sockets
              │   │   │
     ┌────────┘   │   └──────┐
     │            │          │
┌────┴────┐ ┌─────┴───┐ ┌────┴────┐

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [termsurf/termsurf](https://github.com/termsurf/termsurf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
