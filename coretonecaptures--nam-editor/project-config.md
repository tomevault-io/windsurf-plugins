---
trigger: always_on
description: NAM Lab is a desktop metadata editor for `.nam` files (Neural Amp Modeler captures). It lets capture artists view, edit, and bulk-manage the JSON metadata embedded in `.nam` files without touching the model weights.
---

# CLAUDE.md — NAM Lab

## Project Overview

NAM Lab is a desktop metadata editor for `.nam` files (Neural Amp Modeler captures). It lets capture artists view, edit, and bulk-manage the JSON metadata embedded in `.nam` files without touching the model weights.

Built with **Electron + React + TypeScript + Tailwind CSS**, using `electron-vite` for build tooling and `electron-builder` for packaging.

Runs on Windows, macOS, and Linux.

---

## Architecture

### Process Structure (Electron)

```
src/main/index.ts          — Main process: file I/O, IPC handlers, window management
src/preload/index.ts       — Preload script: exposes typed `window.api` to renderer
src/renderer/src/          — Renderer process: all React UI
```

The renderer never touches the filesystem directly — everything goes through `window.api` IPC calls defined in the preload.

### Key IPC Channels

| Channel | Direction | Purpose |
|---|---|---|
| `dialog:openFiles` | main | Open file picker |
| `dialog:openFolder` | main | Open folder picker |
| `file:read` | main | Read and parse a .nam file |
| `file:writeMetadata` | main | Surgically patch metadata in a .nam file |
| `folder:scanNam` | main | Recursively scan folder for .nam files |
| `folder:scanTree` | main | Build folder tree structure |
| `file:move` | main | Move a .nam file to a different folder |
| `path:stat` | main | Check if a path is a directory |
| `shell:revealFile` | main | Open file location in Explorer/Finder |
| `file:trash` | main | Move files to OS trash via `shell.trashItem()` |
| `file:copy` | main | Copy files to destination directory |
| `file:clearNamLab` | main | Surgically remove `metadata.nam_lab` block from files |
| `file:readBinary` | main | Read any file as base64 (used for xlsx import) |
| `dialog:openImportFile` | main | Open file picker for .xlsx/.csv import |
| `window:refocus` | main | Restore keyboard focus after native dialogs |
| `log:getErrorLogPath` | main | Path to parse error log |
| `log:getStartupLogPath` | main | Path to startup log |

### File Write Strategy — CRITICAL

**Never use `JSON.parse` → `JSON.stringify` to write back .nam files.** This destroys formatting and is unacceptable.

All writes use `patchMetadataFields()` in `src/main/index.ts` — a surgical text patcher that finds each changed key in the raw file text using regex and replaces only the value bytes. All original formatting, whitespace, field order, and non-metadata data (weights, config, etc.) are preserved exactly.

For nested fields like `metadata.training.nam_bot.trained_epochs`, use `patchNamBotField()` which navigates into the nested block and creates the structure if it doesn't exist.

For `nl_` fields, use `patchNamLabField()` which writes into `metadata.nam_lab.*`, creating the block if needed.

To remove the entire `nam_lab` block, use `removeNamLabBlock()` which surgically strips `"nam_lab": {...}` including comma handling.

Only fields in `EDITABLE_FIELDS` (plus `nb_trained_epochs`) are ever written — the patcher is a whitelist, not a catch-all.

### Watcher Suppression
`suppressWatcher()` sets `watcherSuppressUntil = Date.now() + 3000`. Any `folder:changed` event fired within 3s of a local write is silently dropped — prevents false-positive "new files detected" banners after saves.

---

## Renderer Structure

### `src/renderer/src/App.tsx`
The root component. Owns all application state:
- `files: NamFile[]` — all loaded files
- `selectedIds: Set<string>` — currently selected file paths
- `librarian: LibrarianState` — folder tree, selected folder, root folder
- `settings: AppSettings` — loaded from localStorage
- `listViewMode`, `treeWidth`, `listWidth` — layout state

Key functions:
- `loadFiles()` — reads files via IPC, applies defaults from settings, tracks `autoFilledFields`
- `loadFolderByPath()` — scans a folder, builds tree, loads all files
- `applyDefaults()` — applies settings rules to metadata at load time
- `handleDrop()` — native document-level drag/drop handler (registered via `useEffect`, not React props)

Layout uses three resizable panels: **FolderTree | FileList | MetadataEditor/BatchEditor/MultiSelectEditor**. Both the tree and file list panels have collapse buttons on their drag handles.

### Components

| Component | Purpose |
|---|---|
| `Toolbar` | Top bar: Open, Save All, Close All, Refresh, Name from File, Settings toggle |
| `FolderTree` | Left panel: folder hierarchy, dirty counts, search/filter, right-click actions |
| `FileList` | Middle panel: list or grid view, filters, column chooser, export |
| `MetadataEditor` | Right panel: single-file editor with all editable + read-only fields |
| `MultiSelectEditor` | Right panel: editor for 2+ selected files, shows shared/varies state |
| `BatchEditor` | Right panel: batch field editor for a folder or selection |
| `SettingsPanel` | Right panel: app settings (replaces editor content when open) |
| `DuplicatesModal` | Full-screen modal: find dupes by filename or metadata name, choose keep, move to _Duplicates or trash |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [coretonecaptures/nam-editor](https://github.com/coretonecaptures/nam-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
