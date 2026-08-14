---
trigger: always_on
description: Pixel icon editor: PySide6 GUI that edits pixels, layers, and animations, and
---

# AGENTS.md

## What this is

Pixel icon editor: PySide6 GUI that edits pixels, layers, and animations, and
exports multi-size `.ico`, `.png`, and `.svg` files. Windows target, Python
3.10+ (3.14 confirmed).

## Run

```bash
pip install PySide6 pillow
python main.pyw       # console visible; use pythonw main.pyw for silent GUI
```

No CI. Tests: `python -m pytest -q`. Lint report: `Errors.cmd` → `Errors.md`.

## Architecture (multi-file — NOT single-file anymore)

- `main.pyw` — entry point only: `from src.app import main`
- `src/app.py` — `MainWindow`: wires panels ↔ canvas, file import/export dialogs,
  shortcuts, recent files, autosave, themes, plugin UI, settings persistence
  (`_restore_settings`/`_save_settings` via `QSettings("PixelEditor", "Settings")`).
  Multiple documents: each tab is a `_Document` (canvas + file + `dirty` flag)
  in a `QTabWidget`; `self.canvas` and `self._current_file` are properties that
  delegate to the active document, so all handlers operate on the current tab
  without call-site changes. `_add_document`/`_close_tab`/`_activate_document`/
  `_sync_ui_to_document` manage tabs; `_apply_window_settings(canvas)` pushes
  the window-level view toggles (grid/mirror/onion/EAL/brush preview/tolerance/
  brush size) onto every canvas; `_wire_canvas` sets the color/stack/modified
  callbacks + event filter per canvas. `canvas._on_modified` fires
  `_mark_dirty()` which sets the active doc's `dirty` flag and refreshes the
  tab/window title with a `*` marker; saving (`_write_project`), loading
  (`_load_file`), and a fresh document clear it. File → Save (Ctrl+S) =
  `_save_current` (Save As for untitled docs), File → Save As = `_save_project`.
  `_close_tab`/`closeEvent`/`_load_file` prompt Save/Discard/Cancel (or
  Yes/No for replace) when a document is dirty.
- `src/canvas.py` — `PixelCanvas(QWidget)`: pixel ops, tools, layers, selection,
  zoom, grid, flood fill, mirror, undo/redo, drag-and-drop. Transforms
  (`rotate_cw`/`rotate_ccw`/`rotate_180`/`flip_h`/`flip_v`) apply to the
  selection content when a selection exists, else the active layer in place
  (canvas dims unchanged; non-square content is centered). All are undoable,
  blocked on a locked layer, and use exact PIL transposes
  (`_transform_qimage` → `qimage_to_pil_rgba` + `pil_to_qimage_exact` in
  `helpers.py`).
- `src/panels.py` — `LeftPanel` (canvas/file/export/edit/view/plugins)
  and `RightPanel` (tools/color/layers/anim); `ColorSwatch`, `ToolButton`
- `src/dialogs.py` — `ResizeDialog` (with 9-point anchor), `ExportPreviewDialog`,
  `ExportDialog` (unified ICO/PNG/per-layer export), `ShortcutOverlay`,
  `AnimationPreviewDialog`, `AnimationExportDialog`,
  `PluginDialog`, `PaletteDialog` (GPL/PAL/hex import-export),
  `QuantizeDialog` (2–256-color reduction, live preview, palette extraction)
- `src/export.py` — `export_ico`, `export_png`, `export_svg`, `batch_export`,
  `export_layers`/`layer_images`/`layer_file_names`,
  `export_animation`/`save_animation` (GIF/APNG), `_warn_upscale`
- `src/project.py` — `save_project`/`load_project` (.icon ZIP: meta.json +
  per-layer PNG), `PROJECT_MAGIC`
- `src/helpers.py` — `qimage_to_pil_rgba`, `pil_to_qimage_rgba`,
  `bresenham_line`, `rect_pixels`, `ellipse_pixels`, `polygon_pixels`,
  `parse_palette_text`/`palette_to_text` (pure, tested)
- `src/plugins.py` — `PluginInfo`, `load_all_plugins()` scans the app
  `plugins/` dir and `%APPDATA%\PixelEditor\plugins\` (via
  `QStandardPaths.AppDataLocation`), accepting `.py`, `.zip`, and
  sub-folders per source; zips stage to `tempfile.mkdtemp`, cleanup runs
  in `MainWindow.closeEvent`. `load_plugins(plugins_dir)` is kept for
  single-directory use.
- `src/constants.py` — `PALETTE_COLORS`, `TOOL_EMOJIS`, `TOOL_CURSORS`,
  `ALL_TOOLS`, `MAX_RECENT_FILES`
- `src/version.py` — single source of truth: `__version__`, `APP_NAME`
  (mirrored in `pyproject.toml`; `tests/test_version.py` enforces the
  match). The window title (`_update_title`) and `AboutDialog` (`Help → About`)
  stamp the version.
- `src/strings.py` — all user-facing text in one flat `dotted.key` table
  (`STRINGS`) plus a `t(key, **fmt)` helper. Call sites in
  `dialogs.py`/`panels.py`/`app.py`/`export.py` look up via `t("...")`.
  Unknown keys fall back to the key itself; placeholders use
  `str.format(**fmt)`. Adding a translation = parallel module with the
  same keys, picked at runtime.

## Key details

- Image format throughout: `QImage.Format_RGBA8888`. QRgb is `0xAARRGGBB`;
  use `_qrgb_from_qcolor` in `canvas.py` for conversion.
- `canvas.image` is a property bound to the **active layer's** image
  (`_layers[active]`). For rendering/export, use `canvas.composite_image()`
  (bottom layer first, honoring visibility + opacity) — `export.py`,
  autosave, and the export preview all use the composite now.
- Dirty tracking: `canvas._on_modified` (a `Callable[[], None]`, set by the app
  in `_wire_canvas` to `_mark_dirty`) fires from `_save_undo_state()`,
  `undo()`, `redo()`, and `set_image_size()` — so every stroke, resize, and
  undo/redo past a save marks the document dirty. Direct `set_pixel_rgba`
  calls (tests, programmatic) do NOT mark dirty.
- Layer-flag toggles are undoable: `toggle_layer_visibility`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [shaheeniptv1/PixelEditor](https://github.com/shaheeniptv1/PixelEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
