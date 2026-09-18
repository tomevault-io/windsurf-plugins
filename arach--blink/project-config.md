---
trigger: always_on
description: Blink is a native macOS spatial note-taking app. It is not a document library
---

# AGENTS.md

## Blink in 60 seconds

Blink is a native macOS spatial note-taking app. It is not a document library
with detachable windows: **the note is the window, and the desktop is the
workspace**.

The human loop is capture → place → recall:

1. Create or find a note from the menubar popover, command palette, or global
   new-note hotkey.
2. The note opens as its one floating `NSPanel`; reopening it focuses that panel
   rather than creating a duplicate.
3. Move, resize, shade, or focus panels to arrange a working set. Blink restores
   each note to the same place.
4. Edit and read in the same panel. The menubar popover and command palette are
   launchers, not alternate document windows.

Blink is an `LSUIElement` menubar-only app: floating note panels + menubar
popover + command palette, with no Dock icon and no main/library window. The v1
Tauri app remains at tag `v1-final`; port lessons and behavior, never its code.

When operating Blink rather than changing its code, use the `blink` CLI for
notes and edit `config.json` for behavior/appearance. Drive the GUI only when a
task specifically requires the live visual surface.

## State model

The markdown files are the durable truth for notes. Runtime note state
converges through one path:

```text
panel edits / popover actions → PanelManager / AppModel → NoteStore → atomic .md
CLI / external writes → same .md → directory watcher → NoteStore.reconcile()
NoteStore notifications → AppModel snapshot → popover / palette / open panels
```

- Notes live at `~/Library/Application Support/Blink/Notes/<id>.md`, where `id`
  is a unique title-derived slug. Body and metadata travel together in each
  markdown file; metadata is YAML frontmatter. Unknown frontmatter must survive
  round trips verbatim. UUIDv5 identity is derived only for v1 compatibility.
- `NoteStore` owns note mutations and the in-memory index. `AppModel` mirrors it
  for UI surfaces; it is not a second store.
- `PanelManager` owns live windows, pending editor text, one-panel-per-note
  identity, geometry, and save flushing. Do not bypass it for panel lifecycle.
- Exact per-device panel frames (`NSWindow` frame autosave), the open-panel set,
  and each note's read/edit mode live in `UserDefaults`. Portable
  `blink.slot` frontmatter is placement intent; the autosaved frame is the
  device-specific position. `NotePanel` suspends frame autosave while physics
  owns a drag/fling/shade and persists the unshaded resting frame once settled.
- External writers are first-class. The directory watcher calls
  `NoteStore.reconcile()`, producing the same notifications as in-app edits.
- `config.json` owns behavior and appearance and hot-applies independently of
  note state. `BLINK_HOME` overrides the file-backed root (notes, config, and
  attachments) for tests/agents; it does not redirect `UserDefaults`. The Mac
  peer host is disabled whenever `BLINK_HOME` is set so sandbox notes can never
  inherit the production device identity or trust list.

## Source-of-truth order

When documents disagree, use this order:

1. Current code and tests describe shipped behavior.
2. This file's hard requirements are non-regression constraints.
3. `docs/cli.md`, `docs/config.md`, `docs/workspaces.md`, and `docs/release.md`
   describe shipped agent-facing or release surfaces.
4. `docs/placement.md` is mixed-status: the grid primitive is shipped; its
   live-plane collision, scene, and tidy semantics are proposed.
5. `docs/v2-plan.md`, `docs/v2-ui-map.md`, and
   `docs/notes-representation.md` are local, untracked design references absent
   from a fresh clone. They contain intent and history and may lag the code.
6. `docs/agent-api.md` and `docs/agent-integration.md` are proposals, not
   implemented surfaces.
7. `docs/functionality-v1.md` is a local, untracked v1 inventory and
   scope/lessons donor, not a description of the v2 codebase.

Other useful references:

- UI studies / visual spec: `design/studio` (tracked shared-Studio app;
  `bun dev` → `localhost:3060/studio`).
- The CLI operates on the same files as the app:
  `blink ls/cat/new/present/type/write/search/rm/path/workspace`; see
  `docs/cli.md`.
- Agents configure Blink by editing
  `~/Library/Application Support/Blink/config.json`, not by driving Settings;
  see `docs/config.md`.
- `blink workspace` defines a named treatment in `config.json`; membership is
  one `blink.workspace` frontmatter key. See `docs/workspaces.md`. Brands are
  generic treatments—never hardcode a specific product identity into Blink.

## Where changes belong

- `Sources/BlinkApp` — AppDelegate/status item/popover, command palette,
  AppModel, PanelManager, NotePanel, config hot reload, and WebBridge.
- `Sources/BlinkCore` — pure Swift with no AppKit: note model and identity,
  frontmatter, atomic file storage, NoteStore, treatments, workspaces, and grid
  math / panel physics.
- `Sources/BlinkCLI` — `swift-argument-parser` CLI over BlinkCore. `BlinkPaths`
  keeps the app and CLI on the same locations.
- `Sources/BlinkPeer` — encrypted Multipeer Connectivity discovery, pairing,
  and snapshot transport shared by macOS and iOS. Bonjour advertises presence;
  new devices request explicit Mac approval inside an encrypted session, and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arach/blink](https://github.com/arach/blink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
