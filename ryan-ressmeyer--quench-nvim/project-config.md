---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quench is a Neovim plugin for interactive Python development that enables cell-based execution similar to VS Code's Jupyter extension. The plugin manages IPython kernels and routes output to both terminal and web browser for rich media display.

**Key Architecture**: Asyncio-based Python application using pynvim, consisting of:
- **Kernel Session Manager**: Manages IPython kernel lifecycles
- **Web Server**: aiohttp server that relays kernel output via WebSockets  
- **Neovim UI Manager**: Handles Neovim API interactions

This is a fully implemented and production-ready plugin with comprehensive testing and complete functionality.

## Development Commands

### Testing
- `pytest tests/` - Run the full test suite
- `pytest tests/unit/` - Run unit tests only
- `pytest tests/e2e/` - Run end-to-end integration tests
- `pytest tests/unit/test_quench_plugin.py` - Run main plugin tests
- `pytest tests/e2e/test_quench_run_cell.py` - Run cell execution tests
- `pytest -v` - Run tests with verbose output
- `pytest -k "test_name"` - Run specific test by name
- The project uses pytest with async support and dependency detection

### Code Formatting
- `stylua --color always --check lua` - Check Lua code formatting
- `stylua lua` - Format Lua code (uses .stylua.toml configuration with 120 column width)
- `.venv/bin/black rplugin/python3/quench/` - Format Python code
- `.venv/bin/black --check rplugin/python3/quench/ tests/` - Check Python code formatting (CI check)
- `.venv/bin/flake8 rplugin/python3/quench/` - Check Python code style

**Note**: This project uses a local uv-managed virtual environment (`.venv/`). Always use `.venv/bin/` prefix for Python linting and formatting commands (black & flake8).

### Plugin Development
- Plugin files are in `rplugin/python3/quench/`
- Main plugin class is in `rplugin/python3/quench/__init__.py`
- Uses pynvim decorators: `@pynvim.plugin`, `@pynvim.command`, `@pynvim.function`
- The Python environment for testing is managed by uv: `/home/ryanress/code/ubuntu-config/nvim/pynvim-env/.venv/bin/python`

## Code Architecture

### Current State
- **FULLY IMPLEMENTED** - All core Quench functionality is complete and working
- Main plugin class in `rplugin/python3/quench/__init__.py` with async commands:
  - `QuenchRunCell` - Execute Python cells with `#%%` delimiters
  - `QuenchStatus` - Display plugin status and active sessions
  - `QuenchStop` - Stop all plugin components
- Complete kernel session management with IPython integration
- Web server with WebSocket relay for rich media output
- UI manager for Neovim API interactions
- Frontend HTML/JS application for browser-based output display

### Additional Commands Available

**Execution Commands:**
- `QuenchRunCellAdvance` - Execute cell and move cursor to next cell
- `QuenchRunSelection` - Execute selected text as Python code
- `QuenchRunLine` - Execute current line
- `QuenchRunAbove` - Execute all cells above current position
- `QuenchRunBelow` - Execute all cells below current position
- `QuenchRunAll` - Execute all cells in buffer

**Kernel Management Commands:**
- `QuenchInterruptKernel` - Send interrupt signal to the kernel (similar to Ctrl+C)
- `QuenchResetKernel` - Restart kernel and clear its state
- `QuenchStartKernel` - Start a new kernel not attached to any buffer
- `QuenchShutdownKernel` - Shutdown a running kernel
- `QuenchSelectKernel` - Select/attach kernel for current buffer
- `QuenchDebug` - Show diagnostic information for debugging

### Key Design Patterns
- Asyncio-based architecture for non-blocking operations
- Central message queue (asyncio.Queue) for component communication
- Cell-based execution using `#%%` delimiters in Python files
- Web browser integration for rich media (plots, audio, etc.)
- Web server auto-starts on VimEnter by default (configurable via `quench_nvim_autostart_server`)
- Automatic kernel death detection and recovery for robust execution

### Kernel Lifecycle Management

The plugin implements three distinct kernel restart/recovery mechanisms:

1. **Manual Restart** (`restart()` method in `kernel_session.py`)
   - Triggered by: User via `QuenchResetKernel` command
   - Behavior: Restarts the kernel process using `km.restart_kernel()`
   - Message sent: `kernel_restarted`
   - Output cache: Preserved (outputs remain visible)
   - Use case: User wants to clear kernel namespace while keeping outputs

2. **Death Detection** (`_monitor_process()` method in `kernel_session.py`)
   - Triggered by: Background monitoring loop (checks every 2 seconds)
   - Detection: Kernel process dies unexpectedly (OOM, crash, external SIGKILL)
   - Behavior: Sets `is_dead` flag, sends `kernel_died` message, cleans up resources
   - Message sent: `kernel_died`
   - Use case: Kernel crashes or is terminated by OS

3. **Auto-Restart on Execution** (`execute()` method in `kernel_session.py`)
   - Triggered by: Attempting to execute code when `is_dead == True`
   - Behavior: Automatically calls `start()` to restart kernel, clears `is_dead` flag, proceeds with execution
   - Message sent: `kernel_auto_restarted`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ryan-ressmeyer/quench.nvim](https://github.com/ryan-ressmeyer/quench.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
