---
trigger: always_on
description: **comfyui_envtools** is a Python GUI tool for managing ComfyUI environments on Windows. It uses CustomTkinter for UI and provides comprehensive Python environment management, plugin dependency checking, and package installation capabilities.
---

# Copilot Instructions for ComfyUI Environment Tools

## Project Overview

**comfyui_envtools** is a Python GUI tool for managing ComfyUI environments on Windows. It uses CustomTkinter for UI and provides comprehensive Python environment management, plugin dependency checking, and package installation capabilities.

**Architecture**: Two-layer separation—backend logic (`comfy_venvtools.py`) and UI presentation (`ComfyUI_CustomTkinter.py`), communicating through callback functions and queues.

## Core Architecture & Data Flow

### Backend (`comfy_venvtools.py`)
The `ComfyVenvTools` class handles all business logic:
- **Mirror/Environment**: Mirrors are defined as `PYPI_MIRRORS` dict; `set_python_env()` and `set_mirror()` cache last selections
- **Package Detection**: `_get_installed_packages_batch()` caches package info for 30 seconds per Python version to avoid repeated `pip freeze` calls
- **Dependency Analysis**: 
  - `check_dependencies()` compares requirements files against installed packages
  - `compute_missing_specs()` returns uninstalled packages in original spec format (e.g., `torch==1.0.0`)
  - Package names are normalized to handle underscore/hyphen variants
- **Installation Flow**: 
  - `simulate_install()` runs `pip install --dry-run` for pre-validation
  - `actual_install()` executes real installations via subprocess with progress callbacks
  - Git operations use standard `git` commands (clone, pull, etc.)

### Frontend (`ComfyUI_CustomTkinter.py`)
The `ComfyUIEnvironmentManager` class manages UI and orchestration:
- **Thread-Safe UI Updates**: All long-running operations use background threads; UI updates go through `_ui_queue` (Queue) and are processed in main thread via `_drain_ui_queue()` every 50ms
- **Event System**: 
  - `_enqueue_text()` → appends to result textbox
  - `_enqueue_progress()` → updates progress bar
  - `_enqueue_deps_values()` → updates dependency file dropdown
- **Configuration**: `config.json` stores Python paths, mirror choice, custom nodes dirs, histories, and dependency caches
- **Three-Column Layout**: 
  - Left: Environment/plugin selection, buttons for various operations
  - Right: Results textbox with scrolling output
  - Bottom: Progress bar

## Critical Developer Workflows

### Running the Application
```powershell
# Activate venv if exists
& .\\.venv\\Scripts\\Activate.ps1

# Run main application
python ComfyUI_CustomTkinter.py
```

### Windows-Specific Handling
- **No Console Window**: All subprocess calls use `CREATE_NO_WINDOW` flag (defined globally in both files)
- **Path Handling**: Use `os.path.normpath()` and backslashes; code accounts for both `\\` and `/`
- **Title Bar**: Dark mode applied via `_set_dark_titlebar()` using Windows DWM API

### Common Operations in Code
1. **Scanning custom nodes**: `_scan_customnodes_async()` calls `tools.scan_customnodes_dependencies()` with progress callback
2. **Dependency checking**: `check_dependencies()` validates requirements files and returns human-readable results
3. **Git operations**: All git interactions use `subprocess` with `CREATE_NO_WINDOW`; git command availability verified before use
4. **File dialogs**: Custom dark-themed dialogs via `_ask_directory_dark()`, `_ask_open_filename_dark()`, etc.

## Project-Specific Patterns

### Environment Root Matching
A critical validation: `_same_environment_root()` checks if Python exe and plugin directory share the same drive + first-level directory. Example:
- ✅ `F:/kontext/python.exe` and `F:/kontext/ComfyUI/custom_nodes` → same root
- ❌ `F:/kontext/python.exe` and `E:/other/ComfyUI/custom_nodes` → different root

This prevents installation mismatches and is enforced before dependency operations.

### Package Name Normalization
Package names use inconsistent formats (e.g., `torch-vision` vs `torchvision`). The code:
- Normalizes via `_normalize_package_name()`: lowercase, replace underscores/hyphens
- Falls back to dual-form checking if batch lookup fails
- Handles both `pip show` and `pip freeze` output formats

### Progress Callbacks
Most async operations accept `progress_cb: Callable[[float], None]` receiving 0.0–1.0 values. Frontend maps this to progress bar updates.

### History Management
Recent selections (Python paths, custom nodes dirs, plugin URLs, library names, commands) are stored in `config.json`. Each "Add to history" method:
1. Removes duplicate (if exists)
2. Inserts at front
3. Limits list to first 20 items
4. Calls `save_config()`

## External Dependencies & Integration Points

### Key Dependencies
- **customtkinter**: Modern GUI framework (dark theme, better styling than tkinter)
- **subprocess**: All external commands (pip, git, robocopy, etc.)
- **json**: Configuration persistence
- **threading**: Background task execution

### Command-Line Tools (Must Be Available)
- `python` / `pip`: Package management
- `git`: Plugin cloning/updating/checking
- `robocopy`: Fast file backup/restore on Windows
- `taskkill`: Force-terminate processes if needed

### Environment Variables

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dxhklck/comfyui_envtools](https://github.com/dxhklck/comfyui_envtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
