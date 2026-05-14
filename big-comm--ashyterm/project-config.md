---
trigger: always_on
description: Guide for AI agents editing this repo. Read once before editing.
---

# AGENTS.md

Guide for AI agents editing this repo. Read once before editing.

## What this is

GTK4/libadwaita terminal emulator for Linux (BigLinux/Manjaro focus). Pure Python via PyGObject. Differentiators: SSH sessions with ControlMaster/ProxyJump, integrated file manager, per-command output highlighting, optional AI assistant, tab groups, split panes.

## Stack

- **Language:** Python 3.8+. Type hints on every public signature.
- **UI:** GTK 4.0, libadwaita 1, VTE 3.91. Also Pango, Gdk, Gio, GLib, Graphene, Secret.
- **Runtime deps:** PyGObject, py7zr (encrypted backup), regex, Pygments, setproctitle, requests, psutil.
- **Build:** `pyproject.toml` (uv-build). Packaging: `pkgbuild/PKGBUILD`.
- **Tests:** pytest, 2267 tests, ~6s full suite. Optional: `hypothesis` for property tests.

## Layout

```
src/ashyterm/
  app.py              Adw.Application entry point.
  window.py           Main window (mixin composition).
  cli_parser.py       CLI args.
  helpers.py          Widget-generic helpers.

  terminal/           VTE core: tabs, splits, SSH/local spawn, highlight,
                      drag-drop, streaming parser.
                      manager.py = orchestrator. pane_manager.py = splits.
  sessions/           Model + sidebar tree. models.py = SessionItem,
                      SessionFolder, LayoutItem. tree.py = ColumnView.
  filemanager/        Local + remote FS (SFTP/rsync). manager.py orchestrates.
                      transfers.py = upload pipeline.
  ui/                 Dialogs, toolbars, menus, actions. actions.py centralizes
                      Gio.SimpleAction. dialogs/ is modular per feature.
  settings/           Persistence + validation. manager.py singleton.
                      config.py has all defaults. highlight_loader.py =
                      output highlight rules.
  data/               Static data: command manager builtins, highlight
                      contexts (51 JSONs under data/highlights/).
  state/              Window state (geometry, open tabs) for restore.
  core/               Event bus (AppSignals), async task helpers.
  utils/              Logger, crypto (keyring), icons, a11y, platform,
                      osc7, ssh_config_parser, security, syntax_utils.
```

**Large files (>1000 lines) — plan a split before editing:**
`terminal/tabs.py` (1304), `terminal/manager.py` (1070), `filemanager/manager.py` (1155), `terminal/_streaming_handler.py` (1033).

## Conventions that diverge from defaults

1. **File size:** target 200-300 lines. Justify anything over 500.
2. **Function size:** 4-20 logical lines.
3. **Type hints:** required on every public Python signature.
4. **Names:** grep-friendly. Avoid `data`, `manager`, `helper`, `util`, `process` when a specific name fits.
5. **Warnings = failures.** Run `pytest tests/ -q` before finishing.
6. **Language split:**
   - End-user UI text + end-user docs: **Brazilian Portuguese**.
   - Code, identifiers, comments, commits, internal docstrings: **English**.

## Known traps (do not repeat)

### Project logger rejects printf-style

`ThreadSafeLogger` in `utils/logger.py` exposes `info(message: str, **kwargs)`. Passing `%s`/`%d` with positional args **raises `TypeError` every call**. Use f-strings.

```python
# BAD — silently breaks inside try/except
self.logger.info("Loaded %d contexts", n)
# GOOD
self.logger.info(f"Loaded {n} contexts")
```

Stdlib `logging.getLogger(...)` (rarely used) accepts printf, but stay consistent with f-strings.

### Gtk.Label alignment

`Gtk.Label()` defaults to `xalign=0.5` (centered). In menus and rows always pass `xalign=0.0` explicitly — otherwise labels re-center after every refactor. Recurring bug.

### CssProvider: modern API

`load_from_data(bytes)` deprecated since GTK 4.12. Use:

```python
provider.load_from_string(css)  # css is str, not bytes
```

### Dialogs: use modern Adw versions

- `Gtk.Dialog`, `Gtk.MessageDialog`, `Adw.MessageDialog` → **`Adw.AlertDialog`** (use `present(parent_widget)`, not `transient_for`/`modal`).
- `Gtk.FileChooserNative`, `Gtk.FileChooserDialog` → **`Gtk.FileDialog`** (async, `*_finish` callbacks).
- `Gtk.FontButton` → `Gtk.FontDialogButton` (`dialog=Gtk.FontDialog()`).
- `Gtk.ColorButton` → `Gtk.ColorDialogButton` (signal `notify::rgba`, not `color-set`).

### Reparent in GTK 4.10+

`Gtk.Paned.set_start_child/set_end_child` and `Adw.Bin.set_child` **assert the new child has no parent**. Always unparent first:

```python
# BAD
grandparent.set_child(survivor)  # still has parent_paned → CRITICAL
# GOOD
if parent_paned.get_start_child() is survivor:
    parent_paned.set_start_child(None)
grandparent.set_child(survivor)
```

### Keyboard shortcuts: lowercase keysym

`Gtk.accelerator_parse("<Control><Shift>Equal")` fails. Use `equal`, `plus`, `minus`, `a`-`z` lowercase. Validator in `settings/validator.py` rejects bad accelerators silently and logs on every settings write.

## Tests

### Run

```bash
pytest tests/ -q                    # full suite
pytest tests/test_foo.py -v         # single file
```

Optional deps: `hypothesis` (property tests), `py7zr` (backup KAT). Install via `pacman -S python-hypothesis` or `pip install -e .[test]`.

### Config isolation


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [big-comm/ashyterm](https://github.com/big-comm/ashyterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
