---
trigger: always_on
description: <!-- SPDX-FileCopyrightText: 2026 Adeveda Enterprises Private Limited -->
---

<!-- SPDX-FileCopyrightText: 2026 Adeveda Enterprises Private Limited -->
<!-- SPDX-License-Identifier: GPL-2.0-or-later -->

# Mixar

**Mixar is a custom fork of Blender 5.0 that turns Blender into an AI-powered 3D content creation tool**, with deep integrations for texture painting, AI-assisted modeling, and real-time agent chat. It's built as an **overlay system** — Mixar's source code gets layered on top of upstream Blender during build.

## Build & Development

### Build Commands
```bash
make init          # Initialize: git submodules + LFS files
make build         # Full build: overlay + CMake + compile + install packages
make clean_build   # Clean build: removes source/ and rebuilds
make install       # Install Python packages into embedded Blender Python
```

### How the Build Works
- `scripts/unix/build.sh` orchestrates the build
- `scripts/unix/overlay.sh` copies `/src` onto `/source` (Blender upstream)
- Config loaded from `.env` → env vars via `scripts/unix/settings.sh` → `scripts/generate_config.py` generates runtime `mixar.json` into the app bundle
- C++ env header auto-generated at `source/creator/mixar_env_config.h`
- Python build-frozen env marker auto-generated at `source/scripts/mixar/config/_build_env.py` (gates `get_dev_bypass_credentials`; `DEV_BYPASS_ALLOWED=True` only when `MIXAR_ENV=Dev`). Setting `DEV_BYPASS_*` env vars with `MIXAR_ENV != Dev` aborts the build.
- Python packages installed from `scripts/python_requirements.txt` into embedded Blender Python
- **Never run `cmake` or `make` directly in `source/`** — always use `make build` or `./scripts/unix/build.sh` (overlay must run first)

### Testing
```bash
pytest              # Run tests (root conftest.py stubs bpy via MagicMock)
```
Tests can run outside Blender. The root `conftest.py` injects `bpy` stubs into `sys.modules`.

## Code Rules

- Keep the code as modular, readable and performance efficient as possible. No file should be larger than 500 lines of code. Leverage C++ wherever needed to get maximum performance.
- All code goes in `/src` — it gets overlaid onto Blender source during build.
- Write all python code inside `/src/scripts/mixar/modules` in the relevant module. When in doubt, ask for the correct module.
- Place all code which can be used across modules in the `common` folder.
- All C/C++ files should be inside `src/source/blender`.
- Put all environment variables in `.env` (copy `.env.example` as template). Never commit `.env`. Config is generated at build time by `scripts/generate_config.py`.
- Keep properties and operators segregated in different folders.
- Keep all the constants for a module inside a `constants.py` file inside the module root.
- **Always update this CLAUDE.md** when features are added, modified, or deleted — keep module descriptions, architecture tables, and patterns in sync with the actual codebase.

## Bootstrap & Registration

The bootstrap system in `src/scripts/startup/bootstrap/__init__.py` handles module loading in 3 phases:

1. **Package setup** — creates synthetic packages for `/src/scripts/mixar/` (no `__init__.py` needed in subdirs)
2. **Bootstrap modules** — loads `src/scripts/mixar/bootstrap/*.py` (must have `register()`/`unregister()`)
3. **UI modules** — auto-discovers all files in `modules/**/ui/` dirs, loads in time-budgeted batches

**Key rules:**
- Only make `register` and `unregister` functions when needed. Let the fallback mechanism handle registrations — pass the list of classes properly via `classes` tuple.
- UI classes in `ui/` are auto-registered by bootstrap. Properties load first (priority 0), then operators/core (1), then panels/menus/headers (2).
- For cross-directory property dependencies, use the module's `__init__.py` to import in order (see `paint/__init__.py`).

## Project Structure

```
.env.example                       # Environment config template (copy to .env for local dev)
scripts/generate_config.py         # Generates runtime mixar.json at build time from env vars
src/
├── scripts/
│   └── mixar/
│       ├── bootstrap/         # Startup modules (agent_connection, paint_module, etc.)
│       ├── config/            # Logging config
│       └── modules/
│           ├── common/utils   # Shared utilities
│           └── {module_name}/
│               ├── constants.py  # Module constants
│               ├── core/         # Functional and calculation logic
│               └── ui/           # UI elements (auto-discovered)
│                   ├── properties/   # PropertyGroup definitions
│                   ├── operators/    # Operator definitions
│                   ├── panels/      # Panel drawing code
│                   ├── menus/       # Menu definitions
│                   └── lists/       # UIList definitions
└── source/
    └── blender/               # C/C++ extensions
```

### Active Modules
`agent_bubble`, `asset_search`, `auth`, `common`, `hunyuan`, `mesh_segment`, `moodboard`, `paint`, `space_mixie`, `space_mixie_chat`, `space_texture_sets`, `texel_density`, `uv_editor`, `workflow`

---

## Core Architecture

| Layer | Location | Language | Purpose |
|-------|----------|----------|---------|
| **Blender C++ modifications** | `src/source/blender/` | C/C++ | Native editor spaces, auth, chat UI rendering |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mixar-AI/mixar-app](https://github.com/Mixar-AI/mixar-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
