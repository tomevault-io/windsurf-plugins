---
trigger: always_on
description: validates, the editor strips as you type.
---

# BindSight — Claude project guide

Star Citizen binding visualizer and mapper. Answers "what does each button
do, and where does each action live?" by joining SC's config with live input
from joysticks, gamepads, keyboard and mouse.
Secondary: the tools the game lacks — editing bindings without starting it,
binding profiles, backups, applying a profile or backup per device, comparing
two binding sets, and fixing the joystick order.

## Design philosophy (the maintainer's, and the top rule)

An app is worthless if it does not follow its intention as well as it can.
BindSight's intention: **as simple and powerful as possible, filling the
gaps SC leaves** (missing features, bad UX). It is an **addon to the game**
and replicates the game's data and behaviour as closely as it can, so the
user never has to translate between the two.

- **Labels, descriptions, tokens: 1:1 with the game.** Only what SC itself
  shows is shown; nothing synthesized. If SC has no label (a `kb1_lalt+x`
  combo), the raw token is the correct display, not an invented "Left Alt +
  X".
- **Behaviour: what SC does.** A combo resolves only while the modifier is
  held, a rebind replaces the kind's binding, defaults come from
  `defaultProfile.xml` — because that is what the game does. Where SC's
  behaviour is unknown, say so; do not guess a nicer one.
- **Before proposing a nicer name or a "helpful" behaviour, check whether
  SC has it.** If it does not, the game's way is the answer. Ask when
  unsure.

## Game-file safety (the second top rule)

The app edits the game's **live** data (`actionmaps.xml`, the profiles
folder), whose structure and processing can change with any game patch.
**BACKUP, BACKUP, BACKUPS**: the user must be able to undo any change at
any time. Concretely:

- **A verified backup before every write** to a live game file; restore is
  byte-exact. Never write without one — except when the user switched
  auto-backups off in Settings, their explicit choice made against a
  warning dialog.
- **Watertight sanitizing** of every user input that reaches a file, a path
  or an XML attribute (names, tokens, ids, paths: no traversal, nothing
  that needs escaping).
- **Rock-solid XML**: parsing tolerates whatever SC or the user may produce
  (comments, CDATA, CRLF, tabs, BOM, self-closing elements, entities,
  single quotes) and never panics; a textual rewrite is re-parsed before it
  touches the disk; writes are atomic (temp file + rename).
- **Tests and safeguards first**: the highest test coverage goes to
  game-file handling (`rebind.rs`, `resort.rs`, `apply.rs`, `backups.rs`,
  `binding_profiles.rs`, `diff.rs`, `scdata.rs` parsing, every writing
  command); every edge case above has a test, added with the change.

## Conventions

- **English only** — all code, comments, commit messages and GUI text.
- **GUI text speaks the player's language**: no file names (`actionmaps.xml`,
  `Game.log`, …) or other internals in labels, titles, toasts and dialogs.
  Only where an element is specifically about that file (the `global.ini`
  override, a missing-/broken-file error detail). Say "game", never "SC"
  ("Star Citizen" is fine where it reads better). Terse: one-word states,
  explanatory sentences only where a dialog guides an action.
- **Repo content policy**: no SC game data in the repo — the app extracts what
  it needs from the user's install at runtime. The `Data.p4k` reader and the
  CryXmlB decoder (`p4k.rs`, `cryxml.rs`) are ports from StarBreaker (MIT,
  by diogotr7): keep the credit header in both files and the notice in
  `THIRD-PARTY-LICENSES.md`.

## Stack

- **Shell/backend**: Tauri v2 + Rust (`src-tauri/`).
- **Frontend**: Vue 3 + TypeScript + Vite (`src/`).
- **Input**: SDL2 raw joystick API (`sdl2` crate) for buttons/axes/hats, its
  GameController API for gamepads, the webview for keyboard keys; `hidapi`
  for the HID product string (SC's device name) and the axis usages.
- **XML/INI**: `quick-xml` + hand-rolled parsing.

## Frontend (`src/`)

`App.vue` is the orchestrator (all state, invokes, listeners) and composes
presentational components:

- `TopBar` — modes **Monitor / Bindings / Devices**, environment chip +
  dropdown, version chip, Refresh, gear. It is also the title bar (see the
  undecorated-window gotcha).
- `StartupTile` — covers every mode while the first game-data load after
  start runs, whatever its outcome, and until the startup image-map load
  is done (`endStartup` waits for `mapsReady`, so the maps never pop in
  after the stage is already showing).
- Monitor: `DeviceTile`, `StatusPanel`, `ImageStage` (+ `Splitter`),
  `LastInputCard`, `BindingsDeck` (flat rows or one bucket per input, same
  head as the Bindings List).
- `BindingsView` — the whole Bindings mode. Left: Game Bindings (the live
  file as the one item "Current"), binding profiles (Save Profile, Import,
  Export), backups (Create Backup). Right: the **Bindings List** for Current
  (the game's keybinding screen as a table, one toggleable column per device
  the file names, categories collapsible, "Set binding" / double-click =
  rebind dialog editing one input at a time, pending rebinds kept until
  Save / Discard in the action tile above it, which also holds Reorder —
  swap two joystick slots the game ranks now, one swap at a time, either

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [w00zla/BindSight](https://github.com/w00zla/BindSight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
