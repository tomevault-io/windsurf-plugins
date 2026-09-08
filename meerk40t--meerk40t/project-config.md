---
trigger: always_on
description: This is the single source of truth for AI coding agents working on MeerK40t.
---

# AGENTS.md - MeerK40t Development Guide

This is the single source of truth for AI coding agents working on MeerK40t.
Other agent instruction files (`CLAUDE.md`, `.github/copilot-instructions.md`)
point here and must not be treated as separate references.

## Project Overview

MeerK40t (pronounced "MeerKat") is an open-source laser cutting/engraving control software. It provides a highly extensible, plugin-based platform supporting multiple laser hardware types including K40 (Lihuiyu), GRBL, Ruida, Moshiboard, Newly, and galvo (Balor) lasers.

**License:** MIT
**Python:** 3.6+
**Platforms:** Windows, macOS, Linux, Raspberry Pi
**Version:** defined as `APPLICATION_VERSION` in `meerk40t/main.py` (avoid hard-coding it here — check the source)

See `NOTES.md` for current device stability status.

---

## Quick Commands

```bash
# Install with all features
pip install meerk40t[all]

# Run application
meerk40t                        # Installed console script (full GUI)
python meerk40t.py              # From a source checkout
python meerk40t.py --no-gui     # Console mode
python meerk40t.py --simpleui   # Simplified interface

# Run tests
python -m unittest discover test -v
pytest -v

# Code quality
flake8 meerk40t test
black --check meerk40t test
mypy meerk40t
```

---

## Architecture

### Plugin-Based System

Everything is a plugin with lifecycle phases:
```
plugins → preregister → register → configure → boot → postboot → start
```

Standard plugin pattern (the `plugin()` function **must** be defined at module level):
```python
def plugin(kernel, lifecycle=None):
    if lifecycle == "register":
        kernel.register("path/to/item", item)
    elif lifecycle == "postboot":
        init_commands(kernel)
    elif lifecycle == "boot":
        # Initialize during boot phase
        pass
```

Lifecycle phase usage:
- `register`: Register providers, services, and formats
- `boot`: Early initialization that doesn't depend on other plugins
- `postboot`: Initialize commands that depend on registered services

### Internal vs External Plugins

**Internal Plugins** (`meerk40t/internal_plugins.py`): Core functionality bundled with MeerK40t, registered during the `plugins` lifecycle phase. To add one, import it and append to the plugins list.

| Category | Plugins |
|----------|---------|
| Core | `core.core`, `device.basedevice`, `network.kernelserver` |
| Drivers | `lihuiyu`, `moshi`, `grbl`, `ruida`, `newly`, `balormk` |
| Hardware support | `rotary`, `cylinder`, `coolant` |
| Image & Fill | `image.imagetools`, `fill.fills`, `fill.patterns` |
| File formats | `dxf.plugin`, `extra.ezd`, `extra.lbrn`, `extra.xcs_reader` |
| Tracing | `extra.vectrace`, `extra.potrace`, `extra.vtracer` |
| Fonts & Shapes | `extra.hershey`, `extra.param_functions` |
| Integration | `extra.inkscape`, `extra.serial_exchange`, `extra.updater` |
| Camera | `camera.plugin` |
| GUI | `gui.plugin` |
| Other | `extra.imageactions`, `extra.outerworld`, `extra.winsleep`, `extra.cag` |

**External Plugins** (`meerk40t/external_plugins.py`): Third-party extensions discovered automatically via the `meerk40t.extension` setuptools entry-point group:

```python
entry_points={
    "meerk40t.extension": [
        "myplugin = mypackage.plugin:plugin",
    ],
}
```

External plugins are disabled with `--no-plugins`, disabled in frozen builds (PyInstaller executables; use `external_plugins_build.py` for hardcoded plugins there), and invalidated if `lifecycle == "invalidate"` returns True.

### Key Layers

1. **Kernel** (`meerk40t/kernel/`) - Service bus, plugin system, signals, channels, settings, jobs
2. **Core** (`meerk40t/core/`) - Element/node tree, operations, cutcode, planning, spooling, units
3. **Device** (`meerk40t/device/`) - Hardware abstraction layer and base device
4. **GUI** (`meerk40t/gui/`) - wxPython interface with AUI docking
5. **Drivers** - Hardware-specific: `grbl/`, `lihuiyu/`, `ruida/`, `moshi/`, `newly/`, `balormk/`
6. **Extra** (`meerk40t/extra/`) - File format parsers, tracing, fonts, parametric shapes, utilities

---

## Directory Structure

```
meerk40t/
├── meerk40t/
│   ├── kernel/          # Service bus, signals, channels, settings, jobs
│   ├── core/            # Element/node tree, planning, cutcode, units, drivers
│   │   ├── node/        #   Node base class and all node type implementations
│   │   ├── elements/    #   Element tree service (management, selection, undo)
│   │   └── cutcode/     #   CutCode data structures and primitives
│   ├── device/          # Hardware abstraction (basedevice.py)
│   ├── gui/             # wxPython UI panels, dialogs, scene rendering
│   ├── grbl/            # GRBL driver
│   ├── lihuiyu/         # K40 (Lihuiyu) driver
│   ├── ruida/           # Ruida driver
│   ├── moshi/           # Moshiboard driver
│   ├── newly/           # Newly driver
│   ├── balormk/         # Balor galvo driver
│   ├── image/           # Image processing and rasterization tools
│   ├── fill/            # Hatch fills (scanline, Eulerian) and wobble patterns
│   ├── tools/           # Geometric algorithms
│   ├── extra/           # File parsers, tracing, fonts, utilities
│   ├── camera/          # OpenCV-based camera integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meerk40t/meerk40t](https://github.com/meerk40t/meerk40t) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
