---
trigger: always_on
description: Guidance for AI coding agents working on TreeSheets, a free-form hierarchical data organizer
---

# AGENTS.md

Guidance for AI coding agents working on TreeSheets, a free-form hierarchical data organizer
(hierarchical spreadsheet) written in C++20 on top of wxWidgets. See `README.md` for the user-facing
overview; this file covers what you need to change the code safely.

## Repository layout

| Path | Contents |
| ---- | -------- |
| `src/` | All source code (≈14k lines, almost entirely headers) |
| `TS/` | User-facing data: `docs/`, `examples/*.cts`, `images/`, `scripts/*.lobster`, `translations/`, `readme*.html` |
| `cmake/` | CMake modules: `Lobster.cmake`, `WxPdfDoc.cmake`, `EmbedFiles.cmake`, `Localization.cmake`, `Packaging.cmake`, `UpdateScriptReference.cmake` |
| `platform/` | Per-OS files: Linux desktop/metainfo/MIME, `lsan.supp`, `toolchain-mingw64.cmake`; macOS `Info.plist`/icon, `toolchain-mingw64.cmake`; Windows `.rc`/icon |
| `.github/workflows/build.yml` | CI: Linux (x64, arm64; .deb and AppImage), Windows MSVC (x64, arm64), macOS (universal), then a release per release marker tag |
| `.claude/skills/treesheets-agent/` | Skill and wire protocol for driving a running TreeSheets over its agent socket |

### Unity build: one translation unit

`src/main.cpp` is the only TreeSheets `.cpp` file (plus `src/lobster_impl.cpp` for the Lobster
bindings, `src/stdafx.cpp` for the MSVC PCH, and `src/macclipboard.mm` on macOS). It defines the global
constants and the `A_*` action enum, then `#include`s every other header in dependency order
inside `namespace treesheets` (the Lobster implementation in `treesheets_impl.h` comes first):

```
image.h text.h cell.h grid.h selection.h encryption.h document.h evaluator.h system.h
wxtools.h tscanvas.h tsframe.h agent_server.h tsapp.h
```

As a result:
- New headers must be added to that include list at the right position. Headers have no
  include guards or includes of their own. They rely on what comes before them.
- All system/wx/std includes go in `src/stdafx.h` (the precompiled header), which also does
  `using namespace std;`.
- Everything is one namespace, so avoid short global names that can clash with Windows
  headers. For example, `TA_*`, `DT_*`, `SW_*` and `WM_*` are macros in `wingdi.h`/`winuser.h`.
  Such clashes build on Linux but break the Windows build.
- Some headers are included inside a struct scope. Don't add namespace-level `inline`
  variables there. Use `static inline` members or function-local statics.

### Core types

| File | Type | Role |
| ---- | ---- | ---- |
| `cell.h` | `Cell` | A node: `Text`, optional `Grid *grid`, colors, style, note, `parent` |
| `grid.h` | `Grid` | 2D array of `Cell`s, layout, rendering, most structural operations |
| `text.h` | `Text` | Cell text, editing, cursor handling, drawing (with `textruns.h` / `bidi.h` for RTL text) |
| `selection.h` | `Selection` | A rectangular selection in a grid, or a text cursor range in one cell |
| `encryption.h` | `Encryption` | Password protected files (Monocypher: Argon2id key, XChaCha20-Poly1305) |
| `document.h` | `Document` | One open file: load/save, undo/redo, rendering entry points, and `Action(int)`, the big dispatcher for every `A_*`/`wxID_*` command |
| `evaluator.h` | `Evaluator` | Cell operations/formulas (see `TS/examples/operation-reference.cts`) |
| `system.h` | `System` (`sys`) | Global state: settings (`sys->cfg`, wxConfig), file loading, fonts, images |
| `tsframe.h` | `TSFrame` | Main window: menus, toolbar, tabs, key and menu event routing |
| `tscanvas.h` | `TSCanvas` | Per-tab drawing surface (paint, mouse, keyboard) |
| `wxtools.h` | | wx helpers, `DrawText`, `TextLayoutCache` (direct Pango on wxGTK3), embedded file lookup |
| `tsapp.h` | `TSApp` | Startup and command-line parsing |
| `agent_server.h` | | Local token-authenticated socket for running Lobster scripts (`-a`) |
| `script_interface.h`, `treesheets_impl.h`, `lobster_impl.cpp` | | Lobster scripting API |

### Common change patterns

- **New command/menu item:** add an `A_*` value to the enum in `main.cpp`, add it with
  `MyAppend(menu, A_FOO, _("&Label"), _("Help text"))` in `tsframe.h` (toolbar button there too
  if needed), and handle it in `Document::Action` in `document.h`. Return a status string, or
  `nullptr`/empty on success, following the surrounding cases. Keyboard shortcuts are appended to the menu
  label (`_("&New") + "\tCTRL+N"`). Linux quirks: `Ctrl+Shift+U` is taken by GTK's Unicode input, and
  `CTRLORALT` means Alt on Linux/Windows and Ctrl on macOS.
- **Modifying the document:** call `AddUndo` on the affected cell/grid **before** changing it,
  as the existing actions do. Then trigger relayout/refresh the same way nearby code does
  (`canvas->Refresh()`, `ResetChildren()`, ...).
- **New setting:** a field on `System`, read in `System`'s constructor with `cfg->Read("name", ...)`,
  written with `sys->cfg->Write(...)` where it changes, and usually a check item in the Options menu.
- **New Lobster builtin:** add a pure virtual to `ScriptInterface` (`script_interface.h`),
  implement it in `treesheets_impl.h`, and register it in `lobster_impl.cpp` with the
  `BUILTIN(name, "args", "types", "returns", "doc")` macro (exposed as `ts.<name>`), with argument validation (`vm.BuiltinError(...)` on invalid input). Then

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aardappel/treesheets](https://github.com/aardappel/treesheets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
