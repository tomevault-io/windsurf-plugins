---
trigger: always_on
description: **Meltdown — Fusion 360 AI Plugin**
---

## Project

**Meltdown — Fusion 360 AI Plugin**

A native Fusion 360 AI plugin that lets users without CAD experience design and iterate CNC-machinable structural parts through conversational interaction — similar to "vibe coding" but for 3D CAD. Users describe what they want in natural language, attach reference images, use @context references, and the AI modifies the current design through a visual agentic loop: modify model → capture viewport → Gemini vision reviews → continue or present to user.

**Core Value:** Users can continuously iterate mechanical parts in their current Fusion 360 design through natural language, without needing to know CAD operations — the AI sees its own work and keeps refining until the result matches the intent.

### Constraints

- **Runtime**: Fusion 360's embedded Python (3.10+), no arbitrary pip install — must bundle or vendor dependencies
- **Threading**: All Fusion API calls on main thread; async AI work must dispatch back via custom events
- **AI Model**: Gemini 3.1 Pro via PydanticAI (vision + reasoning)
- **UI**: Palette-based HTML/JS — no native Fusion dialogs for main interaction
- **Design Mode**: Hybrid design environment (first phase default)
- **No MCP**: AI reasoning is plugin-internal, no external AI orchestration protocol

## Technology Stack

## Languages
- Python 3.x - Fusion 360 add-in implementation and command handling
- JavaScript (ES6+) - Browser-based UI in palette components
- HTML5 - Palette UI structure
- SVG - Icon resources
## Runtime
- Autodesk Fusion 360 (desktop application runtime)
- Windows
- macOS
- Not applicable - No external Python package dependencies
## Frameworks
- Autodesk Fusion API (`adsk.core`, `adsk`) - Core Fusion 360 extension framework
- Fusion Add-In SDK - Python-based plugin system with manifest-driven configuration
## Key Dependencies
- `adsk.core` - Fusion 360 Application API, User Interface API, Command system
- `adsk` - Fusion 360 root module namespace for all Fusion APIs
## Configuration
- `DEBUG` flag in `config.py` - Controls logging verbosity
- Manifest-driven configuration via `meltdown.manifest`
## Platform Requirements
- Autodesk Fusion 360 (with Python API access enabled)
- Python 3.6+ (included with Fusion 360)
- IDE with Python support (VSCode with debugging configuration)
- Target: Autodesk Fusion 360 add-ins folder
- Manifest file: `meltdown.manifest` (JSON format)
- Icon: `AddInIcon.svg`
## Add-in Configuration
- Product: Fusion 360
- Type: Add-in
- Author: (empty - template)
- Description: (empty - template)
- Version: (empty - template)
- Startup behavior: `runOnStartup: false`
- Supported OS: Windows and macOS
- Edit enabled: true
- Icon file: `AddInIcon.svg`
- `meltdown/meltdown.py` - Primary add-in entry point

## Conventions

## Naming Patterns
- Module entry points: `entry.py` (e.g., `meltdown/commands/paletteShow/entry.py`)
- Configuration files: `config.py` (e.g., `meltdown/config.py`)
- Utility modules: `[function]_utils.py` (e.g., `general_utils.py`, `event_utils.py`)
- JavaScript: `[feature].js` (e.g., `palette.js`)
- Package markers: `__init__.py` for package directories
- snake_case for all function names
- Command handlers use descriptive names: `command_created`, `command_execute`, `command_destroy`, `command_validate_input`
- Event handlers use pattern: `[element]_[event]` (e.g., `palette_closed`, `palette_navigating`, `palette_incoming`)
- Utility functions: `[verb]_[object]` (e.g., `add_handler`, `clear_handlers`, `handle_error`)
- Private functions (internal only): prefix with underscore (e.g., `_create_handler`, `_define_handler`)
- Global module constants: UPPERCASE (e.g., `CMD_ID`, `CMD_NAME`, `WORKSPACE_ID`, `PALETTE_ID`)
- Local variables and parameters: snake_case
- Type-annotated parameters in utility functions
- Module-level lists for managing state: lowercase (e.g., `local_handlers`, `_handlers`)
- Type hints used in function signatures where clarity is needed
- Import types from `adsk.core` (e.g., `adsk.core.CommandCreatedEventArgs`)
- Cast command inputs with type annotations for clarity: `text_box: adsk.core.TextBoxCommandInput = inputs.itemById('text_box')`
## Code Style
- No detected linting/formatting tool (no .eslintrc, .prettierrc, .flake8, pyproject.toml)
- Follow PEP 8 style conventions by convention
- Indentation: 4 spaces (Python standard)
- Line length: appears to follow standard conventions without strict enforcement
- Spacing: blank lines between function definitions, logical sections within functions
- Not detected. Code quality is maintained through manual review patterns and adherence to template structure
## Import Organization
- Relative imports use parent directory notation: `from ... import config`, `from ...lib import fusionAddInUtils as futil`
- Star imports used sparingly in `__init__.py` for re-exporting utility functions: `from .general_utils import *`
- Module aliasing for command imports: `from .commandDialog import entry as commandDialog`
## Error Handling
- Try-except blocks in top-level entry points to catch all errors: `try: ... except: futil.handle_error('run')`
- Centralized error handling via `futil.handle_error(name)` utility function
- Error handler logs full traceback: `traceback.format_exc()`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DevriesL/fusion-plugin-meltdown](https://github.com/DevriesL/fusion-plugin-meltdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
