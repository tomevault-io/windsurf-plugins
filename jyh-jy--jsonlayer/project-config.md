---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

JsonLayer — a Flutter **Windows desktop** JSON workspace editor. The user picks a disk directory as a workspace; the app manages a tree of `.json` / `.log` documents inside it, opened as tabs with two interchangeable editor modes (JSON source / object tree).

Windows is the only supported target. Web is impossible as-is (`dart:io` dependency); macOS/Linux are unverified (one hardcoded `Process.run('explorer', ...)` and one manual path-separator heuristic remain).

## Commands

```bash
flutter pub get
flutter run -d windows                       # dev run (windows is the only working device)
flutter analyze                              # must be warning-free before a PR
flutter test                                 # all tests
flutter test test/stores/tab_store_test.dart # single file
flutter test --plain-name "文件夹重命名时更新子标签路径"  # single test by name
dart format lib test
flutter build windows --release              # → build/windows/x64/runner/Release/
```

Installer: compile `installer/json_layer.iss` with Inno Setup (not part of the Flutter build).

Note the app runs as a single window with `titleBarStyle: hidden` — a stale `json_layer.exe` from a previous run can block a rebuild; `taskkill //F //IM json_layer.exe` clears it.

## Layer structure and dependency direction

```
pages/ ──> components/ ──> model/            (UI; no dart:io, no direct file access)
       ──> stores/      ──> services/, model/ (ChangeNotifier state + persistence)
                         ──> services/        (WorkspaceService interface → FileWorkspaceService impl)
       ──> utils/, contants/                  (infrastructure; no business deps)
```

- `services/WorkspaceService.dart` is the **only** sanctioned filesystem boundary. UI code must not call `dart:io` directly for workspace files — go through `WorkspaceStore` → `WorkspaceService`. (Exceptions in tree today: `ThemeStore` writes background images to the app-support dir, and `WorkspaceTree` shells out to Explorer.)
- State is `provider` + `ChangeNotifier`, all four stores injected once in `main.dart` via `MultiProvider`. `ThemeStore` is constructed and `loadFromPrefs()`-awaited **before** `runApp` so `MaterialApp` has a theme on frame one.
- Child components take plain value props + closure callbacks (`content`, `onChanged`, `onSave`, `onNavigate`), never store references. Keep it that way — editors are meant to be store-agnostic.

## Store responsibilities

| Store | Owns |
| :-- | :-- |
| `WorkspaceStore` | file tree (`DocumentItem` root), CRUD delegation, expand/collapse, the `locatePath`/`locateTick` signal |
| `TabStore` | open `DocumentTab` list, active tab, dirty flags, path rewriting after rename |
| `EditorStore` | current document's mode/content (largely superseded by per-tab state on `DocumentTab`) |
| `ThemeStore` | `SkinMode` (light / builtInBg / customBg), theme data, background-image persistence |

`DocumentItem` and `DocumentTab` are immutable with `copyWith`; the tree is rebuilt structurally (`_toggleExpandInTree`) rather than mutated. Preserve this — `WorkspaceStore` relies on identity-free rebuilds.

## Non-obvious invariants (breaking these causes real bugs)

- **`_SkinBackgroundWrapper` root must always be a `Stack`** (`lib/main.dart:177`). Swapping the root node type between `Container`/`Stack` per skin mode unmounts the whole `MaterialApp` subtree and trips `_dependents.isEmpty` / "wrong build scope" assertions.
- **Editor mode switching uses `IndexedStack`, not `TabBarView`** (`RequestResponsePanel`). Both editors stay alive so switching JSON ↔ object never re-decodes the JSON or loses scroll/selection. It listens to the `TabController` itself, not `controller.animation!` (null on first frame).
- **Never assign `TextEditingController.text` in `JsonEditor`.** Its setter resets selection to `collapsed(offset: -1)` and wipes `composing`. Editor content round-trips `onChanged → TabStore → Consumer rebuild → didUpdateWidget`, and that round-trip lands while the user may still be mid-composition — clearing `composing` makes the IME commit the character a second time and throws the caret to the top of the file. Chinese/Japanese/Korean input breaks; ASCII hides the bug because there is no composing range. Use `_syncExternalContent()` (preserves a clamped caret) and bail out early when `_controller.value.composing.isValid`. Locked by `test/components/json_editor_ime_test.dart`.
- **`Ctrl+S` is global**, registered via `CallbackShortcuts` around the whole `Scaffold.body` in `HomePage` so it fires regardless of focus. `Ctrl+F` / `Ctrl+Z` / `Ctrl+L` are editor-local (`Focus.onKeyEvent`); `Ctrl+Z` intentionally prefers the search bar's own `UndoHistoryController` when the search bar has focus.
- **Rename must re-append the original extension.** `WorkspaceTree._showRenameDialog` hides `.json`/`.log` as a non-editable `suffixText`; `_confirmRename` strips any suffix the user typed back in and restores the original. After renaming, `TabStore.updateTabPath(old, new, title)` must be called — it also rewrites descendants via `p.isWithin`, otherwise open tabs point at dead paths.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jyh-jy/JsonLayer](https://github.com/jyh-jy/JsonLayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
