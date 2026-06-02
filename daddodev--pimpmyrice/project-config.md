---
trigger: always_on
description: > PimpMyRice: The overkill theme manager - A Python CLI tool for managing themes, styles, palettes, and modules for system customization.
---

# PimpMyRice Agent Guidelines

> PimpMyRice: The overkill theme manager - A Python CLI tool for managing themes, styles, palettes, and modules for system customization.

## Build/Test/Lint Commands

```bash
# Install dependencies and setup (uses uv)
uv sync --dev
uv pip install -e .

# Run all tests
pytest

# Run a single test
pytest tests/test_theme.py::test_install_module -v

# Run tests matching pattern
pytest -k test_gen

# Linting (check only)
ruff check src/pimpmyrice

# Linting (with auto-fix)
ruff check --fix src/pimpmyrice

# Format code
ruff format src/pimpmyrice

# Type checking (strict mode enabled)
mypy src/pimpmyrice

# Import sorting
isort src/pimpmyrice
```

## Main Components

### Theme System (`theme.py`, `theme_utils.py`)
- **ThemeManager**: Central orchestrator managing themes, styles, palettes, and module execution
- **Theme**: Pydantic model representing a theme with modes (dark/light), wallpapers, and tags
- **Mode**: Contains style configuration for dark/light variants
- **ThemeConfig**: User configuration (current theme, mode selection)
- **Style**: Dictionary of color values and module-specific settings
- Themes support multiple modes, tags for categorization, and per-mode wallpapers

### Module System (`module.py`, `module_utils.py`)
- **ModuleManager**: Discovers, loads, and executes modules from `~/.config/pimpmyrice/modules/`
- **Module**: Pydantic model with metadata and action pipelines (pre-run, run)
- **Actions**:
  - `ShellAction`: Execute shell commands
  - `FileAction`: Template rendering to files
  - `PythonAction`: Execute Python code
  - `IfRunningAction`: Conditional execution based on process checks
  - `LinkAction`: Create symlinks
  - `AppendAction`: Append to files
  - `WaitForAction`: Wait for conditions
- **OnEvents**: Lifecycle hooks for module execution:
  - `module_install`: Run once when module is installed (LinkAction, AppendAction)
  - `before_theme_apply`: Pre-processing actions before theme is applied
  - `theme_apply`: Main theme application actions
  - `after_theme_apply`: Post-processing actions after theme is applied
  - `theme_applied`: Actions triggered when any theme is applied
  - `themes_changed`: Actions triggered when themes list changes
- **ModuleState**: Tracks action pipeline execution state (PENDING, RUNNING, COMPLETED, FAILED, SKIPPED)
- Modules are loaded from `module.yaml` or `module.json` files
- Auto-migration from pre-0.5.0 module format (init/pre_run/run/commands → on_events)

### Color System (`colors.py`)
- **Color**: Core class supporting hex, rgb, rgba, hsl, hsv formats with conversion methods
- **Palette**: Collection of colors (primary, secondary, background, foreground, accent)
- **GlobalPalette**: Extended palette with terminal color mappings
- **Palette Generators**: Algorithmic palette generation from images (dark/light variants in `palette_generators/`)

### Configuration (`config_paths.py`)
- Cross-platform config directory management (Linux XDG, Windows %APPDATA%, macOS Library)
- **PIMP_TESTING** env var switches config to `./tests/files` for isolation
- Key paths:
  - `PIMP_CONFIG_DIR`: `~/.config/pimpmyrice/`
  - `PIMP_CACHE_DIR`: Platform cache directory
  - `THEMES_DIR`: `themes/` subdirectory
  - `STYLES_DIR`: `styles/` subdirectory
  - `MODULES_DIR`: `modules/` subdirectory
  - `PALETTES_DIR`: `palettes/` subdirectory
  - `TEMP_DOWNLOADS_DIR`: Cache downloads directory
  - `THUMBNAILS_DIR`: Thumbnails cache directory

### Template System (`template.py`)
- **Jinja2** integration with strict undefined checking
- `process_template()`: Render template strings
- `render_template_file()`: Render template files with search paths
- `process_keyword_template()`: Evaluate single Jinja2 expressions
- `parse_string_vars()`: Expand variables and user paths (`{{home_dir}}`, `{{module_dir}}`, etc.)

#### Template Variables Available

Templates use Jinja2 syntax: `{{variable_name}}`

**Global Variables (always available):**
- `{{home_dir}}` - User's home directory
- `{{config_dir}}` - OS config directory
- `{{pimp_config_dir}}` - PimpMyRice config directory (~/.config/pimpmyrice/)

**Module Variables (when module_name is provided):**
- `{{module_dir}}` - Path to the module directory
- `{{templates_dir}}` - Path to module's templates subdirectory
- `{{files_dir}}` - Path to module's files subdirectory

**Theme Variables (in theme_dict):**
- `{{theme_name}}` - Name of the current theme
- `{{mode}}` - Current mode (dark/light)
- `{{wallpaper.path}}` - Path to wallpaper image
- `{{wallpaper.mode}}` - Wallpaper display mode
- `{{primary}}`, `{{secondary}}`, `{{background}}`, `{{foreground}}`, `{{accent}}` - Core palette colors
- All terminal colors: `{{color0}}` through `{{color15}}`
- Any custom values from styles

**Template Resolution Flow:**
1. Actions receive a `theme_dict` (AttrDict) containing theme data
2. `parse_string_vars()` merges global paths + module paths + theme_dict
3. Jinja2 processes templates with strict undefined checking
4. Result is returned as a string

**Example Usage:**
```yaml
# In module.yaml run actions
run:
  - action: shell
    command: 'wal -i "{{wallpaper.path}}"'
  - action: file

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daddodev/pimpmyrice](https://github.com/daddodev/pimpmyrice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
