---
trigger: always_on
description: A modular, shell-based statusline for Claude Code CLI that displays real-time development information including context usage, rate limits, costs, git status, and more.
---

# Barista - Claude Code Statusline

A modular, shell-based statusline for Claude Code CLI that displays real-time development information including context usage, rate limits, costs, git status, and more.

## Project Overview

Barista is a **Bash shell script** project (Bash 3.2+ compatible for macOS). It generates a customizable status line that Claude Code displays at the bottom of the terminal. The architecture is modular - each piece of information (git, battery, CPU, etc.) is handled by a separate module in the `modules/` directory.

## Architecture

```
Barista/
├── barista.sh          # Main entry point - loads config, modules, orchestrates output
├── barista.conf        # Default configuration file
├── install.sh          # Interactive installer with arrow key navigation
├── VERSION             # Version tracking (semver)
├── modules/
│   ├── utils.sh        # Shared utility functions (MUST load first)
│   ├── directory.sh    # Current directory module
│   ├── context.sh      # Context window usage with progress bar
│   ├── git.sh          # Git branch and status
│   ├── project.sh      # Project type detection (Node, Rust, Python, etc.)
│   ├── model.sh        # Current Claude model info
│   ├── cost.sh         # Session cost and burn rate
│   ├── rate-limits.sh  # 5-hour and 7-day rate limit tracking
│   ├── time.sh         # Date and time display
│   ├── battery.sh      # Battery percentage (macOS)
│   ├── cpu.sh          # CPU usage
│   ├── memory.sh       # RAM usage
│   ├── disk.sh         # Disk space
│   ├── network.sh      # Network info
│   ├── node.sh         # Node.js version
│   ├── docker.sh       # Docker container status
│   ├── weather.sh      # Weather via wttr.in
│   └── ...             # Additional modules
└── README.md
```

## How It Works

1. **Claude Code calls `barista.sh`** with JSON input containing workspace info, model, context window data, etc.
2. **Configuration is loaded** in order of precedence:
   - Built-in defaults in `barista.sh`
   - `barista.conf` in script directory
   - `$CLAUDE_CONFIG_DIR/barista.conf` (user overrides, defaults to `~/.claude/`)
   - `.barista.conf` in current project directory (per-project overrides)
3. **Modules are loaded** from `modules/` directory (`utils.sh` first, then all others)
4. **Modules are executed** in the order specified by `MODULE_ORDER` config
5. **Output is concatenated** with the configured `SEPARATOR` and printed

## Key Files

### `barista.sh` (Main Entry Point)
- Resolves `CLAUDE_CONFIG_DIR` (env var or defaults to `$HOME/.claude`)
- Defines default configuration values
- Loads configuration files in precedence order
- Loads all module files from `modules/`
- Contains module registry mapping names to functions
- Executes enabled modules and joins output with separator
- Adjusts separator spacing based on `DISPLAY_MODE`

### `modules/utils.sh` (Required Utilities)
- **Theme system**: `apply_theme()` sets icons and indicators based on `COLOR_THEME`
- **Cache system**: File-based caching for expensive operations
- **Safe number handling**: `safe_int()`, `safe_divide()`, `safe_percent()`
- **Status indicators**: `get_status()` (3-level), `get_status_4level()` (4-level for rate limits)
- **Icon handling**: `get_icon()` respects `USE_ICONS` setting
- **Progress bar**: `progress_bar()` generates visual bars
- **JSON helpers**: `json_get()`, `json_get_int()` for safe extraction
- **Display mode checks**: `is_compact()`, `is_verbose()`
- **Logging**: `log_debug()` for debug mode

### `barista.conf` (Configuration)
All settings are documented with comments. Key settings:
- `SEPARATOR` - Character(s) between modules (default: `" | "`)
- `DISPLAY_MODE` - `"normal"`, `"compact"`, or `"verbose"`
- `COLOR_THEME` - `"default"`, `"minimal"`, `"vibrant"`, `"monochrome"`, or `"nerd"`
- `USE_ICONS` - Enable/disable emoji icons
- `STATUS_STYLE` - `"emoji"`, `"ascii"`, or `"dots"`
- `MODULE_*` - Enable/disable individual modules
- `MODULE_ORDER` - Comma-separated list defining display order

## Color Themes

The `COLOR_THEME` setting changes status indicators and icons globally:

| Theme | Status Indicators | Icon Style | Use Case |
|-------|-------------------|------------|----------|
| **default** | 🟢 🟡 🟠 🔴 | Standard emoji (📁 📊 🌿) | General use |
| **minimal** | ◦ ◦ ◦ ● | Geometric (→ ◐ ⎇ ◈) | Clean, subtle look |
| **vibrant** | 💚 💛 🧡 ❤️ | Expressive (📂 🎯 🔀 🧠) | High visibility |
| **monochrome** | [OK] [~~] [!!] [XX] | ASCII text (DIR: CTX: GIT:) | Terminal compatibility |
| **nerd** |     | Nerd Font glyphs | Requires Nerd Font |

Themes are applied via `apply_theme()` in `utils.sh` after all configs are loaded.

## Module Structure

Each module follows this pattern:

```bash
# =============================================================================
# Module Name - Brief description
# =============================================================================
# Configuration options:
#   OPTION_NAME    - Description (default: value)
# =============================================================================

module_name() {
    local input="$1"  # JSON input (for modules that need it)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pstuart/Barista](https://github.com/pstuart/Barista) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
