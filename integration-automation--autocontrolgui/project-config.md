---
trigger: always_on
description: AutoControl (`je_auto_control`) is a cross-platform Python GUI automation framework supporting Windows (Win32 API), macOS (pyobjc/Quartz), and Linux (X11). It provides mouse/keyboard control, image recognition, screen capture, action scripting, and report generation through a unified API.
---

# CLAUDE.md — AutoControl

## Project Overview

AutoControl (`je_auto_control`) is a cross-platform Python GUI automation framework supporting Windows (Win32 API), macOS (pyobjc/Quartz), and Linux (X11). It provides mouse/keyboard control, image recognition, screen capture, action scripting, and report generation through a unified API.

- **Package name**: `je_auto_control`
- **Python**: >= 3.10
- **License**: MIT
- **Author**: JE-Chen

## Architecture & Design Patterns

### Strategy Pattern — Platform Abstraction

`wrapper/platform_wrapper.py` auto-detects the OS and loads the correct backend. All wrapper modules (`auto_control_mouse.py`, `auto_control_keyboard.py`, etc.) delegate to the platform-specific implementation. New platform support is added by implementing the backend interface — no wrapper changes needed.

### Facade Pattern — Unified API Surface

`je_auto_control/__init__.py` re-exports all public functions from wrapper and utility modules, providing a single entry point. Users import only `je_auto_control` and access all features.

### Command Pattern — JSON Action Executor

`utils/executor/action_executor.py` maps string command names (e.g., `AC_click_mouse`) to callable functions. JSON action files define sequences of commands with parameters, enabling recording, serialization, and replay of automation flows.

### Observer Pattern — Callback Executor

`utils/callback/callback_function_executor.py` allows registering callback functions that fire after automation actions complete, supporting event-driven chaining.

### Template Method — Report Generation

`utils/generate_report/` provides HTML, JSON, and XML report generators sharing a common structure: collect test records, format output, write file. Each format implements its own rendering.

## Directory Structure

```
je_auto_control/
├── wrapper/              # Platform-agnostic API (Strategy consumers)
├── windows/              # Win32 backend (ctypes)
├── osx/                  # macOS backend (pyobjc/Quartz)
├── linux_with_x11/       # Linux X11 backend (python-Xlib)
├── gui/                  # PySide6 GUI application
└── utils/
    ├── executor/         # JSON action executor (Command pattern)
    ├── callback/         # Callback executor (Observer pattern)
    ├── cv2_utils/        # OpenCV: screenshot, template matching, video
    ├── socket_server/    # TCP server for remote automation
    ├── shell_process/    # Shell command manager
    ├── generate_report/  # HTML/JSON/XML report generators
    ├── test_record/      # Test action recording
    ├── json/             # JSON action file I/O
    ├── project/          # Project scaffolding
    ├── package_manager/  # Dynamic package loading
    ├── logging/          # Logging
    └── exception/        # Custom exceptions
```

## Development Commands

```bash
# Install dependencies
pip install -r dev_requirements.txt

# Install with GUI support
pip install -e .[gui]

# Run unit tests
python -m pytest test/unit_test/

# Run integration tests
python -m pytest test/integrated_test/

# Build package
python -m build
```

## Feature Delivery Rules

### Every feature must ship both a headless API and a GUI surface

No feature is complete unless it can be driven entirely without the GUI **and** has a corresponding GUI affordance. Concretely:

- **Headless core in `utils/` or `wrapper/`**: all business logic lives in a module with zero `PySide6` imports. Users must be able to `import je_auto_control` and call the feature without ever instantiating a Qt class.
- **Re-export from the package facade**: add the public functions / classes to `je_auto_control/__init__.py` and its `__all__` so `import je_auto_control as ac; ac.<feature>(...)` works out of the box.
- **Executor command coverage**: wire an `AC_*` command into `utils/executor/action_executor.py` so the feature is usable from JSON action files, the socket server, the scheduler, and the visual script builder — all without Python glue.
- **GUI tab or control is a thin wrapper**: the Qt widget must only translate user input into calls on the headless core. It must not contain business logic that would be unreachable headlessly.
- **The top-level package stays Qt-free**: `import je_auto_control` MUST NOT import `PySide6`. The GUI entry point is loaded lazily inside `start_autocontrol_gui()`. Verify with:

  ```python
  import sys, je_auto_control  # noqa
  assert not any("PySide6" in m for m in sys.modules)
  ```

- **Tests cover the headless path**: at least one unit test in `test/unit_test/` must exercise the feature through its non-GUI API with no Qt imports.

Features that are inherently interactive (e.g. region picking with the mouse, template cropping) still count as GUI-only — but they must accept programmatic equivalents (e.g. `screenshot(screen_region=[...])` with explicit coordinates) so scripts can replay the same effect headlessly.

## Coding Standards

### Security First


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Integration-Automation/AutoControlGUI](https://github.com/Integration-Automation/AutoControlGUI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
