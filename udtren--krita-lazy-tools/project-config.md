---
trigger: always_on
description: This file provides guidance to Codex when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex when working with code in this repository.

## What this is

A Krita Python plugin (pykrita) that adds layer management, custom actions, a color/name filter docker, an AI segmentation feature (Florence-2 + SAM2.1), a screen color picker (Windows only), and a user-scriptable widget. It uses the Krita Python API and PyQt5/PyQt6.

## Installation / Development

No build step. To test changes, copy `lazy_tools/` and `lazy-tools.desktop` into Krita's pykrita folder and restart Krita:

- Source: `c:\Users\udtre\Projects\krita-plugin\krita-lazy-tools\`
- Install target: `C:\Users\udtre\AppData\Roaming\krita\pykrita\`

The `.venv` at the project root is for IDE support only — Python and Qt are provided by Krita at runtime.

## Architecture overview

### Entry point

[lazy_tools/\_\_init\_\_.py](lazy_tools/__init__.py) is the pykrita entry point. It:
1. Calls `ensure_config_exists()` to create default config files on first run.
2. Instantiates and registers every `Extension` subclass via `Krita.instance().addExtension()`.
3. `LazyToolsExtension.setup()` registers the docker factory via `addDockWidgetFactory()`.
4. `ScreenColorPicker` is conditionally added only on Windows and only if enabled in config.

### Krita extension patterns

- **Dockers** are registered via a `DockWidgetFactoryBase` subclass. The factory is registered inside `Extension.setup()` (not at module level) because `addDockWidgetFactory` requires Krita to be fully initialised.
- **Actions/shortcuts** are registered by subclassing `Extension` and implementing `createActions(window)`. The action name string must match the `name` attribute in the `.action` XML file under `e_scripts/actions.action`.
- The docker factory is registered in `LazyToolsExtension.setup()`, not at module import time (unlike some simpler plugins).

### Module map

| Module | Responsibility |
|--------|---------------|
| [lazy_tools_docker.py](lazy_tools/lazy_tools_docker.py) | Main docker — collapsible sections wrapping the widget modules |
| [lazy_color.py](lazy_tools/lazy_color.py) | `LazyColorLabel` extension — adds color combo box to Krita's Layer Docker |
| [lazy_color_filter.py](lazy_tools/lazy_color_filter.py) | `LazyColorFilter` extension — shows/hides layers by color label |
| [lazy_segment.py](lazy_tools/lazy_segment.py) | Florence-2 + SAM2.1 AI segmentation pipeline (subprocess-based) |
| [lazy_scripts.py](lazy_tools/lazy_scripts.py) | Script execution extension (legacy/deprecated) |
| [compat.py](lazy_tools/compat.py) | PyQt5/PyQt6 shim — **always import Qt symbols from here** |
| [e_scripts/](lazy_tools/e_scripts/) | Action extensions (one class per action), plus `actions.action` XML |
| [widgets/](lazy_tools/widgets/) | QWidget submodules embedded in the docker sections |
| [dialogs/](lazy_tools/dialogs/) | Settings dialog |
| [config/config_loader.py](lazy_tools/config/config_loader.py) | JSON config load/save + path resolution |
| [utils/layer_utils.py](lazy_tools/utils/layer_utils.py) | `get_current_layer()`, `get_selected_layers()` helpers |
| [utils/color_scheme.py](lazy_tools/utils/color_scheme.py) | Shared `ColorScheme` constants (colours, sizes) |
| [utils/logs.py](lazy_tools/utils/logs.py) | Logging utilities |
| [scripts/](lazy_tools/scripts/) | User scripts auto-discovered and executed by the Scripts widget |
| [scripts_example/](lazy_tools/scripts_example/) | Reference scripts (not executed automatically) |
| [doc/](lazy_tools/doc/) | Internal developer documentation (markdown) |

### e_scripts action modules

Each file in `e_scripts/` defines one `Extension` subclass:

| File | Action |
|------|--------|
| `new_layer.py` | Add new layer with name/type/blend mode options |
| `rename.py` | Rename layer from a predefined name list |
| `duplicate.py` | Duplicate layer with name/blend mode/grouping options |
| `color_pick.py` | Screen colour picker (Win+Shift+C, Windows only) |
| `group_fold.py` | Collapse all group layers |
| `group_expand.py` | Expand all group layers |
| `deselect_alt.py` | Deselect and switch to freehand brush |
| `selection_mask.py` | Create selection mask from active selection |
| `selection_mask_popup.py` | `SelectionMaskPopup` QDialog — shows 64×64 thumbnail grid of all masks in `Selection_Mask_Group`; Create Mask button (calls `create_selection_mask_alt` then refreshes), Refresh button, singleton instance, bold font, top-left grid alignment |
| `set_color.py` | 9 actions to set foreground colour slots 1–9 |

All action names are declared in [e_scripts/actions.action](lazy_tools/e_scripts/actions.action). Every `createActions(window)` call must use the exact `name` string from that XML.

### Widget modules

| File | Docker section |
|------|---------------|
| [widgets/color_filter_widgets.py](lazy_tools/widgets/color_filter_widgets.py) | 8-colour eye-icon visibility toggle grid |
| [widgets/name_filter_widgets.py](lazy_tools/widgets/name_filter_widgets.py) | Prefix/any-match name filter with opacity control |
| [widgets/scripts_widgets.py](lazy_tools/widgets/scripts_widgets.py) | Auto-discovers and runs `.py` files from `scripts/` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [udtren/krita-lazy-tools](https://github.com/udtren/krita-lazy-tools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
