---
trigger: always_on
description: SSH_DeviceManager is a **Tkinter GUI application** for remote device management via SSH/SFTP. The codebase is organized as a Python package (`ssh_device_manager/`) with a thin backward-compatibility launcher (`SSH_DeviceManager.py`).
---

# SSH Device Manager - AI Coding Instructions
# SSH Device Manager - AI Coding Instructions

## Architecture Overview

SSH_DeviceManager is a **Tkinter GUI application** for remote device management via SSH/SFTP. The codebase is organized as a Python package (`ssh_device_manager/`) with a thin backward-compatibility launcher (`SSH_DeviceManager.py`).

### Package Layout

```
ssh_device_manager/          # Main package
    __init__.py              # Re-exports public API
    models.py                # ActionButton, ButtonSection, ToolTip
    ssh_manager.py           # SSHManager (Paramiko wrapper)
    themes.py                # THEMES dictionary (18 built-in themes)
    config.py                # App config / profile persistence
    constants.py             # Shared app constants (limits, file paths)
    paramiko_compat.py       # Clean import when paramiko is absent
    sections_loader.py       # JSON section loading + handler resolution
    validation.py            # Input validation helpers
    output.py                # OutputManager (log queue, append, clear, copy, save)
    app.py                   # SSHGuiApp (Tkinter orchestrator)
    controllers/             # Focused controllers
        connection.py        # Connection lifecycle
        actions.py           # SSH actions and file uploads
        profiles.py          # Profile CRUD
        sections.py          # Section loading, rendering, file watching

SSH_DeviceManager.py         # Thin launcher / backward-compat shim
test_SSH_DeviceManager.py    # 100 unit + integration tests
customizer.py                # Standalone sections.json editor
docs/                        # Test matrix, Gherkin specs, reading guide
```

### Module Responsibilities

| Module | Responsibility | Key Classes/Functions |
|---|---|---|
| `models.py` | UI data structures | `ActionButton`, `ButtonSection`, `ToolTip` |
| `ssh_manager.py` | SSH/SFTP via Paramiko | `SSHManager` |
| `themes.py` | Color theme definitions (18 themes) | `THEMES` dict |
| `config.py` | Profile persistence to JSON | `load_app_config()`, `save_app_config()` |
| `constants.py` | Shared app constants | `COMMAND_HISTORY_LIMIT`, `APP_CONFIG_FILE`, `DEFAULT_SECTIONS_FILE` |
| `paramiko_compat.py` | Safe paramiko import | `paramiko` (real or stub) |
| `sections_loader.py` | Button definitions from JSON | `load_sections_from_file()` |
| `validation.py` | Connection form validation | `get_connection_inputs()`, `parse_int_input()`, `get_host_key_mode()` |
| `output.py` | Thread-safe terminal output | `OutputManager` |
| `app.py` | Tkinter GUI orchestrator | `SSHGuiApp` |
| `controllers/` | Delegated app behaviors | `ConnectionController`, `ActionsController`, `ProfilesController`, `SectionsController` |

## Critical Architecture Patterns

### Thread-Safe Logging with Queue
All network operations run in daemon threads that communicate with the main UI thread via `self.log_queue` (a `queue.Queue`). The `log()` method safely timestamps and queues messages; the log poller runs on the main thread every 80ms to dequeue and display output.

**Example:** `SSHGuiApp.run_ssh_command()` spawns a thread, logs via `self.log()`, not direct output writes.

### Modular Button Sections
Buttons are organized into `ButtonSection` objects. Sections can be defined:
- **Built-in:** via `_define_sections()` in `app.py` (fallback)
- **External JSON:** via `sections_loader.load_sections_from_file()` with handler tokens (`__upload_template__`, `__send_file__`, `__custom_command__`, `run:command`)

Each section has `title`, `max_buttons` (hard limit), and a list of `ActionButton` objects. Disabled buttons are excluded from rendering.

### Theme System
`THEMES` dictionary in `themes.py` maps theme names to color dictionaries. `apply_theme()` in `app.py` modifies `ttk.Style` and raw Tk widgets. Themes are applied dynamically via menu selection.

### Host History (Combobox)
The host field uses `ttk.Combobox` with a special `<Clear History>` option. History persists only during runtime; it's not saved to disk. Capped at 10 entries.

## Key Developer Workflows

### Running Tests
```powershell
python -m unittest test_SSH_DeviceManager.py
```
Tests use `unittest.mock` to mock `paramiko`, `tkinter`, and file dialogs. Tkinter is mocked at import time in the test file. All 100 tests (84 unit + 16 integration) run in under 1 second.

### Adding a New Button
1. In `_define_sections()` in `app.py`, add an `ActionButton` to the appropriate `ButtonSection`
2. Or add to `sections.json` � the app auto-reloads on file changes
3. Set `enabled=True` and provide a handler function
4. If max_buttons is exceeded, excess buttons are truncated with a warning logged

### Extending SSH Commands
Commands execute via `SSHManager.run_command()` in `ssh_manager.py` which uses `client.exec_command()` (non-interactive). For interactive shells (network device paging), override with `client.invoke_shell()`.

### File Transfer
Two methods exist in `app.py`:
- `upload_config_template()`: Fixed remote path (/tmp/uploaded_config.txt)
- `send_file_scp()`: User-specified remote path via dialog

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [esanacore/SSH_DeviceManager](https://github.com/esanacore/SSH_DeviceManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
