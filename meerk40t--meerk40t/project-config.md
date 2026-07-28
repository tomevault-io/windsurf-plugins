---
trigger: always_on
description: **Operating System**: Windows (PowerShell)
---

# MeerK40t AI Coding Agent Instructions

## Environment & Platform Context

**Operating System**: Windows (PowerShell)
- Use native PowerShell cmdlets (e.g., `Remove-Item` instead of `rm`, `Get-ChildItem` instead of `ls`, `Set-Location` instead of `cd`)
- For Python testing, create script files rather than inline execution to avoid PowerShell quoting issues
- When running tests, use: `python -m unittest discover test -v` (PowerShell-compatible)

## Architecture Overview

MeerK40t is a plugin-based laser cutting software built around a **Kernel** ecosystem. The system uses a sophisticated plugin lifecycle system where functionality is dynamically loaded and registered.

### Core Architecture Pattern
- **Kernel** (`meerk40t/kernel/`) - Central service bus providing signals, channels, settings, console commands
- **Core** (`meerk40t/core/`) - MeerK40t-specific ecosystem requirements (elements tree, cutplan optimization, etc.)
- **Device Drivers** (`meerk40t/{grbl,lihuiyu,ruida,moshi,newly,balormk}/`) - Hardware-specific laser control implementations
- **GUI** (`meerk40t/gui/`) - wxPython-based interface with AUI docking framework

### Critical Plugin Lifecycle
All modules follow the plugin pattern with lifecycle hooks. The plugin function must be defined at module level:

```python
def plugin(kernel, lifecycle=None):
    if lifecycle == "register":
        kernel.register("provider/device/grbl", GRBLDevice)
    elif lifecycle == "postboot":
        init_commands(kernel)
    elif lifecycle == "boot":
        # Initialize during boot phase
        pass
```

Lifecycle phases (in order): `plugins` → `preregister` → `register` → `configure` → `boot` → `postboot` → `start`

**Where to add plugin code**: Each module should have a `plugin.py` file (or the main `__init__.py`) that exports the `plugin()` function.

### Internal vs External Plugins

**Internal Plugins** (`meerk40t/internal_plugins.py`):
- Core functionality bundled with MeerK40t
- Registered in `internal_plugins.py` during the `plugins` lifecycle phase
- Examples: core, device drivers, GUI components, image tools
- To add: Import and append to the plugins list in `internal_plugins.py`

**External Plugins** (`meerk40t/external_plugins.py`):
- Third-party extensions loaded via Python entry points
- Entry point group: `meerk40t.extension`
- Automatically discovered at runtime (not in frozen builds)
- To create: Define entry point in `setup.py` or `pyproject.toml`:
  ```python
  entry_points={
      "meerk40t.extension": [
          "myplugin = mypackage.plugin:plugin",
      ],
  }
  ```

**Important**: External plugins are disabled in frozen builds (PyInstaller executables). Use `external_plugins_build.py` for hardcoded plugins in builds.

## Essential Development Patterns

### Device Driver Pattern
Each device type follows this structure:
- `device.py` - Main Device class inheriting from `Service` and `Status`
- `controller.py` - Communication protocol handler
- `driver.py` - Command translation layer (laser operations → device commands)
- `gui/` - Device-specific UI panels
- `plugin.py` - Plugin lifecycle and registration

**Example location**: `meerk40t/grbl/device.py` shows the complete pattern with 1300+ lines of device choices registration.

**How to add a new device driver**:
1. Create a new directory under `meerk40t/` (e.g., `meerk40t/newdevice/`)
2. Implement `device.py`, `controller.py`, `driver.py` following existing driver patterns
3. Create `plugin.py` with registration in the `register` lifecycle phase
4. Register the device provider: `kernel.register("provider/device/newdevice", NewDevice)`
5. Add GUI components in `gui/` subdirectory if needed

### Console Command Registration

Console commands can be registered at the kernel level or service level. Use the appropriate decorator based on context:

**For Kernel-level commands** (available globally):
```python
from meerk40t.kernel.functions import kernel_console_command

@kernel_console_command("command_name", help=_("Description"))
def command_handler(command, channel, _, **kwargs):
    """Long help text goes in docstring"""
    channel(_("Command executed"))
    return "elements", data  # Optional: return context type and data
```

**For Service-level commands** (within a Service class):
```python
from meerk40t.kernel.functions import console_command

class MyService(Service):
    def __init__(self, kernel):
        super().__init__(kernel)
        # Commands are registered automatically via decorator
    
    @console_command("service_command", help=_("Service command description"))
    def my_command(self, command, channel, _, **kwargs):
        """Long help text in docstring"""
        channel(_("Service command executed"))
        # self refers to the Service instance
```

**For Kernel instance commands** (when you have kernel reference):
```python
def init_commands(kernel):
    @kernel.console_command("init_command", help=_("Init command"))
    def cmd_handler(command, channel, _, **kwargs):
        channel(_("Initialized command"))
```

**Key points**:
- `channel` is used for user-visible messages (always use `_()` for translation)
- The second parameter (often `_`) is the remainder string after command parsing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meerk40t/meerk40t](https://github.com/meerk40t/meerk40t) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
