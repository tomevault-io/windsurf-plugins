---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A native desktop player (macOS + Linux + Windows) for the SUB/WAVE internet radio station, built on the **Vercel Native SDK**: declarative `.native` markup + Zig logic, rendered by the SDK's own engine — no browser, no WebView. Requires **Zig 0.16.0** and a global `@native-sdk/cli` **0.10.1** (`npm i -g @native-sdk/cli`).

## Commands

```bash
native build -Dtrace=off && ./zig-out/bin/subwave-desktop   # release build + run
native dev                                       # debug build, Zig hot-rebuild
native test                                      # all tests: unit tests + markup build/layout contract
native check                                     # validate markup + manifest (app.zon)
native package --target macos --output dist      # distributable (also: --target linux)
```

`-Dtrace=off` belongs on every `native build`: the SDK's default trace mode
does unbounded per-frame file I/O on the message loop thread, which is what
stalled it on Windows behind an AV minifilter (issue #23). See
`docs/sdk-notes.md` and `scripts/check-release-flags.sh`.

Headless verification (no human at the screen):

```bash
native build -Dautomation=true -Dtrace=off
./zig-out/bin/subwave-desktop &
native automate wait && native automate screenshot main-canvas
# screenshot lands in .zig-cache/native-sdk-automation/screenshot-main-canvas.png
```

Point the app at a dev station with `SUBWAVE_STATION_URL=http://localhost:<port>` (overrides the persisted station).

## CRITICAL: local SDK patch

The globally installed `@native-sdk/cli` carries **one required local patch** (`patches/native-sdk-local.patch`, documented in `docs/sdk-notes.md`):

- **Fractional HiDPI scale** in `gtk_host.c` — without it the host reports the integer `gtk_widget_get_scale_factor()` instead of the true `gdk_surface_get_scale()`, so on a fractional-scale Linux desktop (e.g. 167%) the canvas is rasterized oversized and nearest-neighbour-resampled down, which shreds glyph antialiasing and makes all text look pixelated. Linux-only; macOS/Windows already read fractional densities.

**After every `npm i -g @native-sdk/cli` upgrade, run `./scripts/apply-sdk-patches.sh` then `native test`.** The script is idempotent and detects partial application.

SDK 0.6.0 absorbed the other two patches (comptime quota; close-hides-window + reserved tray ids 100/101) — do not re-add them. Their replacements are `app.zon`'s `close_policy` plus `fx.showWindow` / `fx.quitApp`; see `docs/sdk-notes.md`.

**`close_policy` is per-build-target, and `app.zon` cannot scope per platform.** The manifest declares `"quit"` (the only thing Linux compiles — the GTK host has no tray to bring a hidden window back), and `scripts/set-close-policy.sh hide` flips it on the macOS and Windows legs of CI and both release workflows. If you touch that line in `app.zon`, keep the `.close_policy = "…"` shape — the script asserts on it and fails the build rather than silently shipping the wrong close behavior.

## Architecture

Elm-style app: a single `Model`, a `Msg` union, and an `update` reducer. All side effects (HTTP, timers, audio, file writes) flow through the SDK **effects channel** — `update` receives `fx: *Effects` and schedules work; results come back as typed `Msg`s. No view code touches I/O.

- `src/main.zig` — thin entry point: shell/window config, `App.create` wiring (`update_fx`, `init_fx`, `tokens_fn`, `view`, `windows_fn`, …), app-level keyboard fallback (`onKey`), tray menu (`statusItem` / `onCommand`), model-declared mini-player window (`windowsFn`), and slider→model sync. Settings load synchronously here *before* the window opens so a saved station skips onboarding.
- `src/model.zig` — the heart (~2300 lines): `Model`, `Msg`, `boot` (init effects), `update` (the reducer, wires every effect), effect keys, settings JSON apply/save. All strings the model keeps are **copied into fixed `*_store` buffers on the Model** — row structs hold slices into those buffers. No heap ownership in the model.
- `src/views.zig` — view registry + composition, and nothing else. The main window dispatches on `model.phase` (onboarding → player); the player is **composed** from six markup fragments (`views/player-top/-sidebar/-stage/-panel/-deck/-sheets.native`), and this file only decides which conditional ones appear. The LIVE stage was hand-built Zig until SDK 0.6.0 added markup's `<image>` leaf (it needs a square runtime image for cover art); there is no Zig view code left. The mini player (`views/mini.native`) is a model-declared secondary window.
- `src/views/*.native` — markup fragments compiled at comptime via `CompiledMarkupView`; a Model field drift is a compile error.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getsubwave/subwave-desktop](https://github.com/getsubwave/subwave-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
