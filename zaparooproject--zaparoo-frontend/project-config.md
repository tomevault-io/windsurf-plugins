---
trigger: always_on
description: Zaparoo Frontend is a Qt/QML frontend for Zaparoo Core. It runs on desktop
---

# Zaparoo Frontend Agent Guide

Zaparoo Frontend is a Qt/QML frontend for Zaparoo Core. It runs on desktop
Linux and on MiSTer FPGA (ARM32, Linux framebuffer, software rendering). The
MiSTer target is the hard constraint: assume no GPU, process kills without
notice, and a small ARM CPU.

Keep this file focused on commands, traps, and rules that are hard to infer
from the tree. Use the docs for longer explanations.

## Commands

Run every workflow from the repo root with `just`. Do not `cd rust/` and run
raw cargo as the default path; the justfile carries the expected environment.

| Task | Command |
|---|---|
| Desktop build | `just build` |
| Desktop run | `just run` |
| Dev run against mock Core | `just mock-core` in one terminal, `just run-dev` in another |
| Full test gate | `just test` |
| QML/C++ tests only | `just test-qml` |
| Rust tests only | `just test-rust` |
| Full lint gate | `just lint` |
| Rust lint only | `just lint-rust` |
| Format | `just fmt` |
| MiSTer ARM32 build | `just arm32` |
| Deploy to MiSTer | `just deploy-mister` |

`just --list` is the source of truth. `CMakePresets.json` and
`rust/.cargo/config.toml` are tuned for those recipes.

## Stack Facts

- Qt 6.7+ with Qt Quick, QuickControls2, QML, QuickTest, and LinguistTools.
- C++17 executable at `src/app/main.cpp`; Rust static library linked through
  Corrosion and cxx-qt.
- Rust workspace is under `rust/`, edition 2021, MSRV 1.90, cxx-qt 0.8.
- Desktop builds link Qt dynamically for LGPL compliance.
- MiSTer ARM32 builds use the Docker toolchain and static Qt.

## Always

- Keep comments and docs in American English.
- After editing C++, Rust, or QML, run `just lint`. Run `just test` when the
  change can affect runtime behavior.
- Keep user-visible state persistent. Selected screen, row/grid positions,
  focus, settings, and similar state must be serialized to disk and restored
  before the first frame. MiSTer's wrapper can kill and relaunch the process at
  any time.
- Wrap user-visible QML strings in `qsTr()` and C++ strings in `tr()`. Use
  `%1`/`%2` placeholders for runtime values so translators can reorder text.

## Ask First

- Before adding or changing a `Client` method in
  `rust/zaparoo-core/src/client.rs`, check the upstream API docs:
  <https://zaparoo.org/docs/core/api/>. Method names, params, and return types
  must match Core.
- Before changing `Sizing.qml` behavior or the persisted state schema, confirm
  the migration/reset behavior.
- Before adding dependencies, changing CI, or touching license/trademark text,
  confirm the intended policy.
- Before changing forward screen routing (`Main.qml` ↔ screens), see the
  "Screens and routing" rules below. Cross-screen Connections and
  per-screen pending flags are how this module bit us last time.

## Never

- Do not use shader-backed or GPU-dependent QML: `LinearGradient`,
  `RadialGradient`, `DropShadow`, `Glow`, `OpacityMask`, `MultiEffect`,
  `Qt5Compat.GraphicalEffects`, Qt Quick Studio shapes, or custom shaders.
  Stick to software-rendering-safe types such as `Rectangle`, `Image`, `Text`,
  `Repeater`, `Item`, `NumberAnimation`, and `ColorAnimation`.
- Do not animate properties that force a large dirty rectangle on busy content:
  no translucent (`opacity < 1`) overlays over a grid, no fading or scaling of
  a parent that contains many delegates, no slide-translation of a band of
  tiles. Qt Software-adaptation cost is dominated by *painted pixels per frame
  × per-pixel cost*, not by the animated property — a fading rectangle over 15
  tiles repaints all 15 tiles per frame, because translucent nodes do not
  subtract from the renderer's obscured region. Pick animations whose dirty
  rect is small (page-dot pulse, focus-ring blink, single-tile move) and let
  the rest of the scene stay static. See `docs/qml-gotchas.md` →
  "Software-renderer animation costs".
- Do not hardcode pixel sizes or fixed element counts in UI. Use
  `Sizing.pctH()`, `Sizing.pctW()`, `Sizing.fontSize()`,
  `Sizing.visibleCovers`, and `Sizing.cornerRadius` (for any rounded-square
  surface — see `docs/style.md`). Any value that drives `x`/`y`/`width`/
  `height`, border widths, margins, or font sizes must go through
  `Sizing.px()`, `Sizing.stroke()`, `Sizing.center()`, or `Sizing.half()`.
  The whole app must run cleanly at 240p; fractional geometry is a bug
  everywhere, not just on MiSTer.
- Do not center user-visible text via `anchors.horizontalCenter` +
  `Text.AlignHCenter`. Center the `Text` item itself with
  `Sizing.center()` and render its glyphs left-aligned (or pre-measure
  with `TextMetrics`). Glyph runs that straddle a half-pixel soften
  under any 240p rendering and aren't acceptable on any screen. See
  `docs/qml-gotchas.md` → "Integer-pixel rules".
- Do not add Qt5 compatibility code or `#if QT_VERSION` guards. This project is
  Qt 6.7+ only.
- Do not change `BUILD_SHARED_LIBS`. Desktop needs `ON`; the ARM32 toolchain
  sets static linking for MiSTer.
- Do not publish state with `tokio::sync::broadcast` when late subscribers need
  the current value. Use `tokio::sync::watch` for state and reserve broadcast
  for lossy events.
- Do not inline a `watch::Sender::borrow()` or any read guard in an `if let`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZaparooProject/zaparoo-frontend](https://github.com/ZaparooProject/zaparoo-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
