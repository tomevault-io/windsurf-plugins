---
trigger: always_on
description: ofs-ng is a funscript editor: libmpv video playback paired with a multi-axis timeline, a live 3D
---

# Engineering Guide

ofs-ng is a funscript editor: libmpv video playback paired with a multi-axis timeline, a live 3D
simulator, a node-based processing graph, and a C# plugin system. See **[README](README.md)** for the
user-facing overview, platform support, and full build prerequisites.

This file is the **engineering contract** for the codebase — the architecture and the rules any change
must respect. It is written for everyone who edits the code, human contributors and AI coding assistants
alike (Claude Code reads it automatically). The rules below are not style preferences: most encode an
invariant that the threading model, the undo system, the localization pipeline, or the plugin ABI
depends on. When in doubt, follow them literally.

## Documentation map

Deeper references live in `docs/`:

| Doc | Covers |
|-----|--------|
| `docs/ARCHITECTURE.md` | The three-primitive design in prose, the interaction extension points, and the processing-node model |
| `docs/TRANSLATING.md` | Maintaining translation catalogs and adding languages via `tools/translations.py` (the `sync`/`todo`/`apply` workflow) |

## Build

See **[README](README.md)** for prerequisites and first-time setup. The commands below use
`cmake-build-debug-visual-studio` (the IDE-generated build directory) — substitute your own configured
directory, e.g. `build`:

```
cmake --build cmake-build-debug-visual-studio -j 8
```

- The no-target build above builds everything; the app itself is the `ofs-ng` target (e.g.
  `cmake --build cmake-build-debug-visual-studio --target ofs-ng -j 8`).
- Warnings are errors (`/WX` on MSVC); clang-tidy runs per-target if found (also warnings-as-errors).
- Format the tree: `cmake --build cmake-build-debug-visual-studio --target format`.

## Test

Build first, then run the whole suite:

```
ctest --test-dir cmake-build-debug-visual-studio --output-on-failure
```

CTest registers four tests: `unit` (no window), `plugins` (PluginManager + real CoreCLR; the .NET
runtime is **mandatory** — a CoreCLR/plugin test that can't init the host **fails**, it never skips),
`ui-smoke` (full window + imgui_test_engine), and `ui-smoke-loc` (the same UI suite re-run under a
machine translation to catch lost `###id`s). Run one with `-R`, e.g.
`ctest --test-dir cmake-build-debug-visual-studio -R ui-smoke --output-on-failure`. To run a single UI
suite or test, pass a filter straight to the binary: `bin/test/ui-tests/ofs-ui-tests --test-filter=plugin_crash`.

## Architecture

The codebase is structured around three primitives. No ECS framework is used.

### Three primitives

| Type | Role |
|------|------|
| `ScriptProject` | Single source of truth for all project state — a plain C++ struct hierarchy |
| `EventQueue` | The only channel for cross-system communication — typed, deferred, thread-safe |
| Services | Behavior units that read `ScriptProject` and push events |

`OfsApp` owns all three and composes them. No framework governs the structure.

### ScriptProject

`src/Core/ScriptProject.h` — owns `AxisState axes[kStandardAxisCount]` (indexed by the `StandardAxis`
enum) plus all top-level state structs: `state`, `overlay`, `simulator`, `videoPlayer`, `metadata`,
`bookmarks`, `playback`, `timelineView`, the always-sorted `regions` vector, the opaque per-plugin
`pluginData` store (round-trips with save; the host never interprets it), per-chapter scene-view memory,
the transient graph-load / auto-eval flags, and the `active*` / `stored*` interaction-selection ids. The
header is the source of truth for the full field list and each field's transient/serialization notes; the
non-obvious invariants are below.

**Rules:**
- **MUST** pass `ScriptProject` as `ScriptProject&`. Never store a `ScriptProject*`.
- **MUST** mutate axis fields only through `ScriptProject::mutate(role, fn, eq)`. This applies `fn`, syncs the selection (removes entries no longer in `actions`), sets `axis.dirty = true`, and pushes `AxisModifiedEvent`. Pass `affectsData=false` for a **display-only** flag write (`isVisible`/`isLocked`/`showInStrip`): the change still persists (dirty + `editRevision`) but skips the `AxisModifiedEvent` push, so it does not kick off a processing re-eval / plugin notify the action data never warrants. It does **not** snapshot undo state — `UndoSystem` auto-snapshots by registering `on<E>()` handlers for all undoable event types before `ProjectManager` does (registration order is preserved by `EventQueue`). The undoable action-mutation events the `EditIntentRouter` emits (`AddActionAtTimeEvent`, `MoveActionEvent`, `RemoveActionAtTimeEvent`, `RemoveSelectedActionsEvent`, `PasteActionsEvent`, `MoveSelectionTime/PositionEvent`) carry a `bool snapshot` field that the router **host-stamps** via a per-gesture latch — `true` on the first mutation of a gesture, `false` on the 2nd..Nth — so a multi-mutation `Replace` (even one mixing adds/removes) coalesces into a single undo step. Each defaults to `true`, so a standalone (non-router) push snapshots on its own. `ModifyRegionEvent` carries the same flag but is UI-set `true` only on gesture start. See *Interaction extension points* in `docs/ARCHITECTURE.md`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ofs69/ofs-ng](https://github.com/ofs69/ofs-ng) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
