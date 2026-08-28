---
trigger: always_on
description: Rules for AI agents working in this repo. Humans: this is also the engineering
---

# Agent rules

Rules for AI agents working in this repo. Humans: this is also the engineering
guide, so read it before a second change. Naming and tone rules are in
[CLAUDE.md](CLAUDE.md) and apply everywhere.

## Layout

- `src/` — the frontend: TypeScript, React + zustand, Vite. Two entry points:
  `overlay/main.ts` (the transparent full-screen window that draws the wand;
  deliberately framework-free, one canvas and a few pointer handlers) and
  `main.tsx` (the spellbook, a React app). Inside:
  - `api/` — `tauri.ts` is the only place that calls `invoke`/`listen`, typed
    per command; `dialog.ts` is the native "choose an application" dialog and
    `caption.ts` the window controls the custom title bar needs, both kept
    separate so importing them is a choice (the overlay has neither);
    `types.ts` mirrors the Rust structs; `mock.ts` stands in for Tauri when
    `vite` runs alone, so the UI previews in a browser with no Rust toolchain.
  - `state/` — zustand stores. `app.ts` (platform, spellbook, wand mode, status),
    `forge.ts` (the spell being drawn/edited), `recorder.ts` (the key-chord
    recorder — a vanilla store, because the overlay is the only thing left that
    uses it; see "Two ways to name a chord" below).
  - `components/` — one file per piece of UI: `Spellbook`, `Forge`,
    `SpellForm`, `SettingsSheet`, `KeyRecorderButton`, `ChordPicker`,
    `CaptionButtons`, …
  - `lib/` — pure helpers with unit tests (`keys`, `chord`, `system`, `color`,
    `path`, `geometry`). `chord.ts` is the one with a counterpart in Rust:
    every key it offers has to be one `shortcut.rs` can press.
  - `wand/` — `wand.ts` is the sprite and trail both windows share;
    `replay.ts` animates a saved rune being redrawn.
- `src-tauri/src/` — the app. `lib.rs` is the wiring: windows, tray, hotkey,
  commands, the worker thread. Behaviour lives beside it in one file per
  concern: `hook.rs` (global keyboard hook state), `recognizer.rs` (`$1`),
  `spells.rs` (spellbook file), `shortcut.rs` (string → key presses),
  `win.rs` (the Win32 calls, the counterpart of the `objc2` blocks in
  `lib.rs`: foreground window, `ShellExecute`, shell icons, elevation).
- `src-tauri/vendor/rdev/` — a vendored copy of `rdev` 0.5.3 with local
  patches, pulled in as a path dependency. Every patch carries a
  `// PATCHED (wandful): …` comment saying why, and grepping for that string is
  the list. Two on macOS: drag events are delivered (`LeftMouseDragged` /
  `RightMouseDragged` / `OtherMouseDragged` in the tap mask and the event
  match), and key events never resolve their Unicode name on the tap thread
  (see "macOS threading"). Two on Windows, both in the hook installers:
  `grab` / `listen` pump messages in a loop instead of once (a single
  `GetMessageA` lets the thread end, and Windows then unhooks without telling
  anyone), and `grab_keys` / `listen_keys` install the keyboard hook without
  the mouse one. A patch stays applicable to upstream's own configurations —
  keep the `#[cfg]`s that guard the code you are patching.
- `src-tauri/capabilities/` — the whole list of host APIs the web views may
  call: `default.json` for both windows, `main.json` for what only the
  spellbook window may do (the title-bar controls). Adding a Tauri plugin call
  means adding a line here — to the narrower of the two where that works — and
  saying so in `SECURITY.md` if it widens what the app can reach.
- `scripts/` — icon and README media generators (`make-*.mjs`, run by hand,
  output is committed), and the two macOS signing helpers.

## The one rule about names

The product is **Wandful**. A shortcut is a *spell*, the set is the
*spellbook*, running one is a *cast*, the trigger gesture is a *swish*. Use
those in code identifiers, user-facing text, docs and commit messages. Do not
introduce a synonym or a codename; the last one (`magic-wand`) has been
removed and should not come back.

## macOS threading

This is where the hours have gone.

- **The `rdev` tap runs on its own thread. Nothing that touches TSM /
  HIToolbox / Cocoa may run there.** Resolving a key's Unicode name does, and
  traps with `SIGTRAP` on recent macOS — that is the vendored patch in
  `vendor/rdev/src/macos/common.rs`. Key synthesis (`enigo`) does too, which
  is why every cast goes through `press_on_main` in `lib.rs`. If you need
  something from the tap thread, send it over a channel and let the main
  thread act.
- **`app.run_on_main_thread` is the way onto the main thread**, not a Grand
  Central Dispatch call of your own. It queues into Tauri's run loop, which is
  the same one AppKit uses.
- **The overlay is hidden, not click-through, while the wand is away.**
  `set_wand_mode` in `lib.rs` shows it and takes focus when the wand comes
  out, remembers the frontmost app's pid, and hands focus back after a cast.
  All mouse handling happens inside the overlay web view while it is up.
  A regression here looks like "the app ate my right click" or "my shortcut
  went to Wandful instead of the app", and is the first thing to check.
- **Escape has three meanings, decided in `hook.rs`.** Recording a shortcut
  → it ends the recording (reported as an `Escape` chord). The overlay's

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ostapondo/wandful](https://github.com/ostapondo/wandful) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
