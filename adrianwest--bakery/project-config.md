---
trigger: always_on
description: KiCad plugin to localize all KiCad symbols and footprints. Built in Python using KiCad's pcbnew/eeschema APIs.
---

# Bakery - KiCad Plugin - AI Coding Agent Instructions

## Project Overview
KiCad plugin to localize all KiCad symbols and footprints. Built in Python using KiCad's pcbnew/eeschema APIs.

**Purpose**: Automate the process of copying global library symbols and footprints into project-local libraries, eliminating external dependencies.

**Tech Stack**: 
- Python 3.x (KiCad's embedded Python)
- KiCad pcbnew API (for PCB/footprints)
- KiCad eeschema API (for schematics/symbols)
- wxPython for UI dialogs

## Architecture
```
Bakery KiCad Plugin
├── Plugin Entry Point (ActionPlugin interface)
│   └── bakery_plugin.py
├── Footprint Localizer (processes footprints and 3D models)
│   └── footprint_localizer.py
├── Library Manager (creates/manages local libraries)
│   └── library_manager.py
├── S-Expression Parser (parses KiCad file format)
│   └── sexpr_parser.py
├── UI Components (logging and configuration dialogs)
│   └── ui_components.py
├── Backup Manager (file backup utilities)
│   └── backup_manager.py
└── Constants (configuration and messages)
    └── constants.py
```

### Major Components
- **Plugin Core** (`bakery_plugin.py`): Main ActionPlugin class registered with KiCad, orchestrates the localization process
- **Footprint Localizer** (`footprint_localizer.py`): Scans PCB/schematics, copies footprints and 3D models to project
- **Library Manager** (`library_manager.py`): Creates/updates fp-lib-table, manages library metadata
- **S-Expression Parser** (`sexpr_parser.py`): Parses and serializes KiCad S-expression format
- **UI Components** (`ui_components.py`): Logger window with progress bar, configuration dialog
- **Backup Manager** (`backup_manager.py`): Creates timestamped backups before file modifications
- **Constants** (`constants.py`): Centralized configuration values and UI messages

## Project-Specific Conventions

### Code Style
- Follow PEP 8 for Python code
- Class names: `PascalCase` (e.g., `BakeryPlugin`, `SymbolLocalizer`)
- Functions/methods: `snake_case` (e.g., `localize_symbols`, `copy_footprint`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `PLUGIN_VERSION`, `DEFAULT_LIB_NAME`)
- KiCad API objects use KiCad's naming (e.g., `GetBoard()`, `GetFootprints()`)

### Docstring Format (Doxygen Style)
Use Doxygen-style docstrings for all modules, classes, functions, and methods:

**File/Module Level Docstring:**
```python
"""!
@file example_file.py

@brief A brief description of the file.

This file demonstrates how to document a Python script using Doxygen-compatible
docstrings. It includes documentation for the module itself, a function, and parameters.

@section description_main Detailed Description
More detailed description of the file and its purpose.

@section notes_main Notes
- Any special notes or considerations.
"""
```

**Function Docstring:**
```python
def copy_footprint(source_path, dest_path, footprint_name):
    """
    @brief Copy a footprint from global library to local project library
    
    @param source_path: Absolute path to source .kicad_mod file
    @param dest_path: Absolute path to destination .pretty folder
    @param footprint_name: Name of the footprint to copy
    
    @return True if copy successful, False otherwise
    
    @throws IOError if source file not found
    @throws PermissionError if cannot write to destination
    """
    pass
```

**Class Docstring:**
```python
class DoxygenExample:
    """!
    @brief A simple example class to demonstrate Doxygen documentation in Python.

    This class provides a basic illustration of how to format docstrings
    for integration with the Doxygen documentation generator.

    @section methods Methods
    - :py:meth:`~DoxygenExample.__init__`
    - :py:meth:`~DoxygenExample.calculate_sum`

    @section attributes Attributes
    - value1 (int): The first value.
    - value2 (int): The second value.
    """
    pass
```

**Doxygen Tags to Use:**
- `@file` - File name (use at top of file)
- `@brief` - Short description (first line)
- `@param` - Parameter description
- `@return` - Return value description
- `@throws` / `@exception` - Exceptions that may be raised
- `@note` - Additional notes or warnings
- `@see` - Cross-references to related functions/classes
- `@section` - Define sections within documentation

### Error Handling
- Use try/except for KiCad API calls (can fail if board/schematic not loaded)
- Show user-friendly wx.MessageBox dialogs for errors
- Log detailed errors to KiCad's scripting console
- Never crash KiCad - always catch exceptions at plugin boundaries

### Testing Strategy
- Manual testing within KiCad required (no easy unit test framework)
- Test with various KiCad project structures
- Verify symbol/footprint integrity after localization
- Test on both Windows and Linux (if applicable)

## Development Workflows

### Setup
```bash
# Install to KiCad plugins directory

# Windows (Quick Install):
install.bat

# Windows (Manual):
# Copy to %USERPROFILE%\Documents\KiCad\9.0\scripting\plugins\Bakery

# Linux:
cp -r Bakery ~/.kicad/9.0/scripting/plugins/

# macOS:
cp -r Bakery ~/Library/Preferences/kicad/9.0/scripting/plugins/

# Restart KiCad to load the plugin
```

### Running the Project
```bash
# The plugin runs within KiCad:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AdrianWest/Bakery](https://github.com/AdrianWest/Bakery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
