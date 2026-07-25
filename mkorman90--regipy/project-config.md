---
trigger: always_on
description: > OS-independent Python library for parsing offline Windows registry hives
---

# CLAUDE.md - regipy

> OS-independent Python library for parsing offline Windows registry hives

## Project Overview

regipy is a forensic-focused library for parsing Windows registry hive files (files with REGF header). It's designed for digital forensics and incident response (DFIR) workflows, providing both a Python API and CLI tools.

### Core Capabilities

- Parse offline registry hives without Windows dependencies
- Recursive traversal of keys and values from any path
- Transaction log recovery (dirty hive reconstruction)
- Hive comparison/diffing (like RegShot)
- Extensible plugin system for artifact extraction
- Timeline generation for forensic analysis

## Architecture

```
regipy/
├── registry.py          # Core RegistryHive class - entry point for all parsing
├── structs.py           # Binary struct definitions (REGF header, NK/VK records, etc.)
├── hive_types.py        # Hive type constants (NTUSER, SYSTEM, SOFTWARE, SAM, etc.)
├── exceptions.py        # Custom exceptions (RegistryKeyNotFoundException, etc.)
├── utils.py             # Helpers: convert_wintime, boomerang_stream, etc.
├── recovery.py          # Transaction log application logic
├── plugins/
│   ├── plugin.py        # Base Plugin class - all plugins inherit from this
│   ├── utils.py         # run_relevant_plugins() - auto-detects hive and runs matching plugins
│   ├── ntuser/          # NTUSER.DAT plugins (persistence, typed_urls, user_assist, etc.)
│   ├── system/          # SYSTEM hive plugins (computer_name, shimcache, bam, bootkey, etc.)
│   ├── software/        # SOFTWARE hive plugins (installed_programs, profilelist, etc.)
│   ├── sam/             # SAM hive plugins (local_sid, etc.)
│   ├── security/        # SECURITY hive plugins
│   ├── amcache/         # Amcache.hve plugins
│   └── usrclass/        # UsrClass.dat plugins (shellbags)
├── cli.py               # CLI entry points
regipy_tests/
├── data/                # Test hive files (often .xz compressed)
├── validation/          # ValidationCase framework for plugin testing
docs/
└── PLUGINS.md           # Plugin development guide
```

## Key Classes and Patterns

### RegistryHive

The main entry point. Handles hive parsing, key navigation, and value retrieval.

```python
from regipy.registry import RegistryHive

reg = RegistryHive('/path/to/NTUSER.DAT')

# Navigate to a key
key = reg.get_key(r'Software\Microsoft\Windows\CurrentVersion\Run')

# Get values
values = key.get_values(as_json=True)

# Iterate subkeys
for sk in key.iter_subkeys():
    print(sk.name, sk.header.last_modified)

# Recursive traversal
for entry in reg.recurse_subkeys(as_json=True):
    print(entry)

# Control sets (SYSTEM hive)
for path in reg.get_control_sets(r'Control\ComputerName\ComputerName'):
    # Yields: ControlSet001\Control\..., ControlSet002\Control\..., etc.
    pass
```

### Plugin System

Plugins inherit from `Plugin` base class and define:
- `NAME`: Snake_case identifier
- `DESCRIPTION`: Human-readable description  
- `COMPATIBLE_HIVE`: Hive type constant from `hive_types.py`
- `run()`: Extraction logic, appends results to `self.entries`

```python
from regipy.hive_types import NTUSER_HIVE_TYPE
from regipy.plugins.plugin import Plugin

class MyPlugin(Plugin):
    NAME = 'my_plugin'
    DESCRIPTION = 'Extract something useful'
    COMPATIBLE_HIVE = NTUSER_HIVE_TYPE
    
    def run(self):
        try:
            key = self.registry_hive.get_key(r'Software\MyKey')
            for value in key.get_values(as_json=self.as_json):
                self.entries.append(value)
        except RegistryKeyNotFoundException:
            pass  # Key doesn't exist in this hive
```

### Timestamp Handling

Always use `convert_wintime()` for Windows FILETIME conversion:

```python
from regipy.utils import convert_wintime

timestamp = convert_wintime(key.header.last_modified, as_json=True)
# Returns ISO format string when as_json=True, datetime object otherwise
```

### Transaction Log Recovery

```python
from regipy.recovery import apply_transaction_logs

apply_transaction_logs(
    hive_path='/path/to/NTUSER.DAT',
    transaction_log_path='/path/to/NTUSER.DAT.LOG1',
    restored_hive_path='/path/to/recovered.DAT'
)
```

## Hive Types

Defined in `hive_types.py`:

| Constant | Typical Files |
|----------|---------------|
| `NTUSER_HIVE_TYPE` | NTUSER.DAT |
| `SYSTEM_HIVE_TYPE` | SYSTEM |
| `SOFTWARE_HIVE_TYPE` | SOFTWARE |
| `SAM_HIVE_TYPE` | SAM |
| `SECURITY_HIVE_TYPE` | SECURITY |
| `USRCLASS_HIVE_TYPE` | UsrClass.dat |
| `AMCACHE_HIVE_TYPE` | Amcache.hve |
| `BCD_HIVE_TYPE` | BCD |

## CLI Tools

- `regipy-parse-header` - Display hive header, validate checksums
- `regipy-dump` - Export entire hive to JSON (or timeline with `-t`)
- `regipy-plugins-run` - Auto-detect hive type and run relevant plugins
- `regipy-diff` - Compare two hives, output differences to CSV
- `regipy-process-transaction-logs` - Apply transaction logs to recover dirty hive

## Development Guidelines

### Adding a New Plugin

1. Create file in appropriate `plugins/<hive_type>/` directory
2. Inherit from `Plugin`, set `NAME`, `DESCRIPTION`, `COMPATIBLE_HIVE`
3. Implement `run()` method - append results to `self.entries`
4. Use try/except for `RegistryKeyNotFoundException` (key may not exist)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mkorman90/regipy](https://github.com/mkorman90/regipy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
