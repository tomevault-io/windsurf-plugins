---
trigger: always_on
description: Defi remains macOS-native, Niri-inspired, and scrolling-columns only.
---

# Defi Agent Notes

Defi remains macOS-native, Niri-inspired, and scrolling-columns only.

## Golden rule

Keep Defi fast, deterministic, stable, and glitch-free.

- avoid unnecessary Accessibility writes
- skip unchanged frames
- prevent layout feedback loops
- preserve per-monitor isolation
- normalize platform events before state mutation
- keep commands and layout testable without Accessibility permission

## Architecture boundaries

- `DefiModel`: pure data and command parsing
- `DefiCore`: pure layout engine
- `DefiConfig`: TOML parsing, defaults, validation, app rules
- `DefiRuntime`: reducer and workspace routing
- `DefiIPC`: Unix-socket protocol
- `DefiMacOS`: AppKit, Accessibility, CoreGraphics, hotkeys
- `DefiDaemon`: daemon wiring
- `DefiCLI`: `defi` command

Never import AppKit, ApplicationServices, or CoreGraphics from pure modules.

## Private platform API policy

Private macOS APIs fall into two categories:

- optional read-only metadata may be used when it materially improves a
  user-visible result and a public-API fallback remains fully functional
- private mutation requires evidence that public APIs cannot meet a
  user-visible correctness, stability, or latency invariant

- isolate private API use inside `DefiMacOS` behind a narrow backend interface
- resolve private symbols dynamically; missing or changed symbols must never prevent startup
- always keep a fully functional, tested public-API fallback
- probe mutating private capabilities using Defi-owned surfaces, never by mutating user windows
- downgrade a mutating private backend for the rest of the session after failure
- fall back immediately when optional private metadata is unavailable or invalid
- expose active private mutation backends and fallback counts through status or telemetry
- third-party position and size mutation is approved only for Defi's
  experimental frame backend after explicit user opt-in; keep it disabled by
  default, while focus, lifecycle, Spaces, and compositor control remain on
  public macOS APIs

## Product shape

Keep:

- scrolling columns
- virtual workspaces
- per-monitor isolation
- keyboard and CLI control
- minimal config by default

Exclude from MVP:

- BSP layouts
- native macOS Spaces control
- compositor replacement
- mouse-first shell

## Runtime rules

All state mutation passes through `DefiRuntime`.

Run exactly one `defi-daemon` instance per user session. Multiple daemons create
competing event taps, socket ownership, AX writes, and visible layout glitches.

- acquire the per-user instance lock before creating the IPC socket
- use `defi service restart` for an installed build; never also use `open -n`
- before replacing an installed bundle, preserve its code-signing identity; set
  `DEFI_CODESIGN_IDENTITY` in the ignored `.env.local` when needed and verify
  the replacement has the same designated requirement, because macOS treats a
  changed requirement as different code and prompts for Accessibility and
  Screen Recording again
- stop the current instance before replacing the installed app bundle
- after build/run verification, confirm exactly one `defi-daemon` process remains

Managed tiled windows fill vertical workspace space. Width changes reflow siblings. Inactive workspace windows park offscreen. Active-workspace focus stays explicit.

Poll-based discovery is acceptable for MVP. Keep cadence bounded and frame writes diffed. AX notifications may replace polling later without changing pure runtime contracts.

## Navigation, focus, and parking invariants

Scrolling navigation must remain speculative and latest-wins.

Preserve these outcomes without freezing a specific implementation:

- stale asynchronous completions must never restore an older focus, frame, layout, or visibility state
- observed real frames and logical targets must converge without feedback loops; neither optimistic targets nor delayed native events are authoritative in every situation
- keyboard capture and command intake must remain responsive while Accessibility or layout work is pending
- scrolling animation must remain monotonic, refresh-aware, and free from abrupt time-based catch-up jumps
- parking must converge and self-repair after delayed application behavior without leaking parked windows into visible monitor regions

Current strategy may evolve when replacement preserves the same outcomes and tests:

- classify AX latency dynamically per process with stable transitions; never hardcode slow-app behavior for Xcode or any other application
- coalesce transient focus changes during rapid navigation; commit real AX focus only for the final target
- keep focus writes asynchronous so slow applications cannot block command intake
- keep horizontal navigation position-only unless a replacement proves synchronous size work cannot enter the input path
- keep vertical workspace transitions position-only and all-or-nothing; use an
  immediate switch when any participating AX lane or display topology cannot
  animate safely within the refresh budget
- settle latency-sensitive windows outside the speculative path while preserving their real targets, hidden state, and parking state

The scrolling workspace is one continuous horizontal strip.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qeude/Defi](https://github.com/qeude/Defi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
