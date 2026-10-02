---
trigger: always_on
description: Guide for agents (and people) working in this repository. It holds the
---

# AGENTS.md

Guide for agents (and people) working in this repository. It holds the
architecture's reasons, the contracts that must not move, how to verify a
change, and the pitfalls that have already cost something. Every pitfall below
was actually hit once; none is a guess.

When this file and the code disagree, the code wins — then fix this file. A
rule written here with no counterpart in the code means one of the two is lying.

## What this is

Evlat is a macOS status strip that sits on the edge of the screen and tells you,
peripherally, what your AI coding sessions are doing. A mascot at the head of
the bar shows the aggregate state; below it, one indicator per session.

The app does not know about "AI sessions". It knows about **`Signal`s**.
Session tracking is the first provider of that abstraction; usage windows,
chat jobs and external commands enter the same way.

## Layout

```
Package.swift
CHANGELOG.md         release notes, `## x.y.z` per version; shown on the release
                     page and in the update window
Sources/EvlatCore/   pure core: Foundation + Dispatch only
Sources/EvlatApp/    AppKit + SwiftUI shell; the NWListener transport lives here
Sources/Evlat/       main.swift — classifies argv (app, `watch`, `signal`, help)
Tests/EvlatCoreTests/
Tests/EvlatAppTests/
Tests/Fixtures/      fake `claude`, fake `ssh`
Resources/{en,tr}.lproj/Evlat.strings
docs/media/          README's banner and screenshots; not bundled into the app
scripts/bundle-app.sh   builds build/Evlat.app; the only source of Info.plist and
                        of the signature (ad-hoc, or EVLAT_SIGN_IDENTITY) and
                        the version (EVLAT_VERSION, EVLAT_BUILD)
scripts/make-appcast.sh writes Sparkle's one-item appcast for a release
scripts/release-notes.sh prints one version's section of CHANGELOG.md
scripts/make-icon.swift draws the app icon; no image is checked in
Makefile
```

Swift 5 language mode, macOS 14 minimum (`PhaseAnimator` and
`KeyframeAnimator` come from there). One third-party dependency: Sparkle, the
shell's updater (`Updater.swift`, the only file that imports it); the core
never sees it.

## Architecture

Two layers, one hard seam. In one sentence: **the core does not import UI.**

```
┌──────────────────────────────────────────────────────┐
│  EvlatApp  (AppKit + SwiftUI)                        │
│  NSPanel · bar geometry · rings · detail card        │
│  mascot · chat bubble · settings · setup             │
└──────────────────────┬───────────────────────────────┘
                       │  seam: Signal ↓  /  Action ↑
┌──────────────────────┴───────────────────────────────┐
│  EvlatCore  (Foundation + Dispatch only)             │
│  Provider · Signal · Registry · Snapshot             │
│  local HTTP API (routing, parsing, defenses)         │
└──────────────────────────────────────────────────────┘
```

### Core rules

- **`EvlatCore` imports only `Foundation` and `Dispatch`.** No `AppKit`,
  `SwiftUI` or `Network`. A test fails if one does. The HTTP route table,
  parsing, dispatch and browser defenses are in the core and tested without
  sockets; only the `NWListener` *transport* is in the shell
  (`HookListener.swift`).
- **Platform capabilities are injected** through `Platform` (liveness, process
  start time, clock). A direct Darwin call from the core is a bug even when it
  compiles.
- **Paths are parameters**, never constants (`~/.claude/sessions` is the
  provider's argument).

This is free discipline, not infrastructure: macOS is the only target today,
but a core that obeys these rules should compile elsewhere; only the UI would
be rewritten.

### The seam: `Signal` and `Action`

Every provider reduces to one type, `Signal`: `provider`, `entity`, `kind`
(`session | usage | job | custom`), `phase`, optional `progress`, `label`,
`detail`, `source`, `fidelity`, `rawStatus`, `updatedAt`, `activity`,
`usage`, `machine`.

- **`Phase` has five values** — `idle`, `working`, `waiting`, `review`,
  `failed` — and stays at five. A new value must update three places at once:
  `Phase.priority`, the bar's indicator language and the mascot's expression
  table; miss one and the new state is silently invisible.
- Every row has a **layer** (`Registry.Layer`, derived in
  `Registry.Snapshot`; not a `Phase`, not a `Signal` field): `waiting`,
  `working`, **news** (a `review`/`failed` whose `Finish` — entity, phase,
  stamp — the user has not seen) and `passive` (a seen finish, or `idle`). The
  first three are active. The list sorts by layer, news newest finish first;
  dimmed rows stay at the bottom and paint nothing.
- `Registry.Snapshot.aggregate` reduces the live, active rows to the
  mascot's one face, by `Phase.priority`: `waiting > news (newest) > working > idle`. With no active
  row the face is `idle`. The seen set is the snapshot's pure input; the core
  keeps no clock and no seen state.
- An unrecognised source word stays **visible** in `rawStatus` and lands in the
  provider's `unrecognizedStatuses`; it is drawn as `idle` but never swallowed.
- `activity`, `usage` and `machine` are not phases and never change priority.
  Usage signals are split out by `kind` and never reach the mascot, the rings

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evlat/evlat](https://github.com/evlat/evlat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
