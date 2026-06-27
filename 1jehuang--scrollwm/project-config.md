---
trigger: always_on
description: Guidance for AI agents (and humans) working in this repo.
---

# AGENTS.md — ScrollWM

Guidance for AI agents (and humans) working in this repo.

## What this is

ScrollWM is a scrolling/PaperWM-style window manager for macOS. Windows live in
columns on a horizontal strip; navigation teleports the viewport. The product
binary is `WindowLab` (the `run` subcommand is the production app; other
subcommands are the lab/test harness). The installed app bundle is
`~/Applications/ScrollWM.app`.

## GOLDEN RULE: test in the sandbox, never against the user's real windows

This tool moves the user's real, live windows. When iterating, **do not arrange
the user's actual session**. Use sandbox mode, which runs the REAL production
controller but is hard-locked to disposable windows it spawns:

```bash
swift build
.build/debug/WindowLab sandbox 4      # spawn 4 throwaway windows, arrange them
```

Why it is safe (see `Sources/WindowLab/Sandbox.swift`):
- `ScrollWMController.sandboxPIDs` forces EVERY arrange path (menu, hotkey,
  direct call) through that PID filter, and the `LifecycleMonitor` only
  observes/adopts those PIDs. The user's real windows are never enumerated or
  moved.
- `RestoreStore.subdirectory` is redirected to `ScrollWM-Sandbox/`, so the
  sandbox's crash-recovery file can never clobber/recover the real session.
- Ctrl-C / Quit restores and terminates the spawned windows.

Only `WindowLab cycle` and a bare `WindowLab run` + manual Arrange touch real
windows. Avoid those while developing; prefer `sandbox`.

## Testing / verification

The integration tests are **HEADLESS BY DEFAULT**: they run the REAL engine +
controller logic against an in-memory window world (`SimWindowWorld`, installed
as `AXSource.backend`). Nothing is spawned, moved, focused, or closed on your
real desktop, and no global keystroke is ever injected, so they are safe to run
anytime, even while you work. Pass `--live` to exercise the old real-window path.

```bash
.build/debug/WindowLab unittest       # pure logic: strip ops, ResyncPlanner, config (no AX)
.build/debug/WindowLab headlesstest   # ALL integration suites, headless (see below)
.build/debug/WindowLab opstest        # width/move/close/focus-sync (headless; --live = real AX)
.build/debug/WindowLab e2etest        # real controller + synthetic chords (headless; --live = real keys)
.build/debug/WindowLab revealtest     # "Arrange All" reveals + adopts hidden/minimized (headless; --live)
.build/debug/WindowLab spawnlatency   # new-window adoption latency (headless; --live = real AX)
.build/debug/WindowLab displaytest    # multi-display placement/parking/rebind (headless 2-display; --live)
.build/debug/WindowLab sandbox [n] [--display M]
                                      # live, interactive, isolated to spawned windows
                                      #   (--display M tiles them on monitor M, 0-based L->R)
```

`make test` runs `unittest` + `animtest` + `headlesstest`. Always run it before
claiming a change is done; it never touches your real windows or focus.

### How headless works (the seam)

`AXSource` and `CGWindowSource` route every window read/write/action through an
optional `WindowBackend` (`AXSource.backend`). In production it is `nil` and the
exact prior C-API path runs. A test installs a `SimWindowWorld` that models the
parts the engine actually depends on: per-window CFEqual-stable element tokens,
app size MINIMUMS (`setSize` below the floor clamps yet returns `.success`), the
current-Space on-screen list (minimized/app-hidden windows drop out), the macOS
off-screen clamp (parked-window sliver lands on the nearest display), system
keyboard focus, and `kAXWindowCreated`/`Destroyed` events for the fast-adopt
path. `HotkeyManager`/`KeyboardEventTap` skip real registration under a backend
and expose `debugDeliver`, and `ScrollWMController.debugDeliverChord` routes a
synthetic chord through the same tap->Carbon precedence as a real keypress, with
NO CGEvent posted. The menu-bar status item is suppressed too. When you change
behavior, prefer extending the sim + a headless assertion over a `--live` test.

Add a test for any behavior you change; prefer extracting pure functions (e.g.
`ResyncPlanner`, `viewportTarget`) so logic is unit-testable without AX.

## Architecture (Sources/WindowLab/)

- `TeleportEngine.swift`     strip model, viewport, teleport commits (focus/move/width)
- `StripOps.swift`           width/move/close ops on the focused column
- `LifecycleMonitor.swift`   keeps strip in sync: AX observer + NSWorkspace + 2s poll
- `WindowEventObserver.swift` AXObserver on kAXWindowCreated -> fast adoption
                             (fixed delay ~8ms coalesce + progressive fast-adopt
                             retry; lands right-of-focus the instant WindowServer
                             publishes the window)
- `ResyncPlanner.swift`      PURE Space-aware adopt/drop policy (unit-tested)
- `AXSource.swift`           timeout-protected AXUIElement wrapper +
                             `WindowBackend` seam (nil in prod; sim in tests)
- `CGWindowSource.swift`     WindowServer enumeration (current-Space = on-screen list)
- `IdentityMatcher.swift`    AX<->CG fusion (PID+frame+title scoring)
- `RestoreStore.swift`       crash-recovery frame persistence

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [1jehuang/scrollwm](https://github.com/1jehuang/scrollwm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
