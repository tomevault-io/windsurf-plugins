---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A native macOS tiling window manager: a direct port of Hyprland's `CHyprDwindleLayout`, with
Omarchy's defaults (`preserve_split = true`, `force_split = 2`). It runs as a background agent
(`.accessory`) with no Dock icon and no main window — a menu bar item, key bindings and a
gradient border around the focused window. Accessibility is the only permission it asks for by
default; the opt-in `[animations] slide_on_swipe` is the one feature behind a second one (Screen
Recording — see `ScreenSnapshot`).

## Commands

```sh
make test      # layout suite — no permissions, no Xcode, ~1s
make run       # build, bundle, sign, relaunch in place (replaces any running toe)
make bundle    # build/Toe.app, runs `make test` first
make install   # same, into /Applications
make dev-cert  # once per machine — see "Accessibility and code signing" below
make reset-perms
make example-config   # regenerate toe.example.toml from the default baked into the binary
```

`swift build -c release` and `swift run -c debug toe-selftest` are what those wrap.

**Running one test:** the harness has no filter. `Sources/toe-selftest/Harness.swift` is a plain
executable rather than XCTest (XCTest ships with Xcode, not the Command Line Tools, so this keeps
`make test` working on a bare CLT machine). The whole suite is ~600 assertions and runs in about a
second, so run all of it. Failures print as `test name:line — what: got X, want Y`.

**Diagnostics without launching the agent:**

```sh
toe --version                # what the bundle was stamped with
toe --print-default-config   # the shipped default TOML
toe --print-corner-radius    # what macOS rounds each on-screen window to
log stream --predicate 'subsystem == "com.clifmeister.toe"' --level info
```

## The two-target split

`Package.swift` enforces the architecture:

- **`Sources/ToeCore`** — pure geometry and state. No AppKit, no Accessibility, no Carbon. The
  dwindle port, workspaces, config parsing, the menu model, session snapshots, border geometry.
  This is the only part the selftest can reach, so **anything worth testing belongs here.** When a
  decision in `Sources/toe` needs test coverage, the move is to lift the arithmetic into ToeCore
  and leave the system call behind — `BorderGeometry` exists entirely for that reason.
- **`Sources/toe`** — everything touching the system. AX, CGWindowList, Carbon hotkeys, event
  taps, AppKit panels.

Adding an AppKit import to ToeCore breaks the test suite's reason for existing.

## How a change reaches the screen

`Coordinator` is the hub and the only `WindowTrackerDelegate`. Roughly:

1. `WindowTracker` discovers windows per-application via `AXObserver` and calls back
   (`windowAppeared`, `windowFocused`, `windowFrameChangedExternally`, `windowStackChanged`,
   `activeSpaceChanged`, `screensChanged`).
2. `Coordinator.dispatch(_ command:)` mutates `WorkspaceManager` — the layout model.
3. `apply(refocus:)` calls `workspaces.render()` for a `RenderPlan` (`frames`, `floating`,
   `stashed`, `focus`) and writes only what differs from `desired`.
4. `updateBorder()` last.

Two invariants in that loop are easy to break:

- **`desired` / `corrections`.** Chromium, Electron and JetBrains apps re-apply their own geometry
  after a window opens, so a frame is re-asserted when it changes behind toe's back — bounded at
  three attempts, so an app whose minimum size exceeds its tile is written once and left alone.
  Floating windows deliberately bypass this machinery: pointing it at one would fight the user's
  own drags.
- **A window the user has hold of is never written to.** `draggedWindow` is checked in `apply`,
  `windowFrameChangedExternally` and `updateBorder`; the frame is written once, on release.

## The quick menu mirrors Omarchy's tree

`MenuModel` is a port of `default/omarchy/omarchy-menu.jsonc` on Omarchy's `quattro` branch, not a
menu that borrowed some names — the same rows at the same depth in the same order, so that an
Omarchy user diving it finds their own menu with the Linux taken out. The rule for what is absent
is Omarchy's own `when` guard: a row whose condition fails is not listed, and a submenu whose
visible descendants have all gone goes with them. Before adding a row, check where upstream puts
it; before leaving one out, check that it genuinely cannot work here rather than that it was
inconvenient.

Two divergences are deliberate and should stay:

- **`Quit` is a root row.** Omarchy's `System` is a power menu for the machine; toe quits an
  agent, and one row does not want a level.
- **`About` is a `.note` row carrying the version**, where upstream's opens a branding window.
  toe has no window to open and one fact to report.

`disabled` (dim, ticked, unselectable, omitted from search) is upstream's guard for "you already
have this" and is why `Install › Style › Theme` can list the whole catalogue. `MenuState` enforces
it in four places — `move`, `select`, `activate` and the search filter — so a new way of reaching a
row needs a fifth.

## Things that will bite you


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theclifmeister/toe](https://github.com/theclifmeister/toe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-05 -->
