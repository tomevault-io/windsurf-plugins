---
trigger: always_on
description: This file provides guidance to Claude Code when working with the VeloIQ™ framework repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with the VeloIQ™ framework repository.

## Repository Layout

```
backend/
  veloiq_framework/         # Framework Python package (pip-installable as veloiq-framework)
    auth/                   # Built-in auth module (User, Role, Tenant + JWT + RBAC)
    extension.py            # VeloIQExtension base class — subclass in extension packages
    extensions.py           # discover_extensions() — scans veloiq.extensions entry points
    scaffold/               # Templates used by `veloiq new <app>` to create new projects
      backend/              # Backend scaffold (main.py, requirements.txt, .env.example)
      frontend/             # Frontend scaffold (App.tsx, allModels.gen.ts, vite.config.ts)
    cli/
      new_extension.py      # veloiq new-extension — scaffold a new extension package
      add_licensing.py      # veloiq add-licensing — scaffold license module into host app
  pyproject.toml            # Framework package metadata (name, version, dependencies)
  setup.py                  # Setuptools compatibility shim

packages/
  ui/                       # @juicemantics/veloiq-ui — React component library
    src/                    # TypeScript source
    dist/                   # Built output (committed — consumers install from this)

docs/                       # Framework documentation

samples/
  task-manager/             # Complete reference application built with the framework
    backend/                # FastAPI backend using veloiq-framework
    frontend/               # React frontend using @juicemantics/veloiq-ui
```

## Commands

### Framework UI package
```bash
cd packages/ui
npm install
npm run build    # Rebuild dist/ after source changes
npm run dev      # Watch mode
```

### Sample task-manager app
```bash
# Backend
cd samples/task-manager/backend
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
pip install -e ../../..   # install veloiq-framework from local source
python seed_sqlite.py     # seed the SQLite database
uvicorn app.main:app --reload
# API: http://localhost:8000 | Docs: http://localhost:8000/docs | Admin: http://localhost:8000/admin/

# Frontend
cd samples/task-manager/frontend
npm install
npm run dev      # Dev server at http://localhost:5173
```

### Project explorer (interactive TUI)
```bash
veloiq          # run with no arguments from inside a project directory
```
Opens a curses TUI showing all modules, models, fields, relations, dashboard
configuration, and search configuration. Supports launching CLI commands
(`add-dashboard`, `search add-model`, `generate`, `add-module`, etc.) with
Y/N confirmation. Implemented in `backend/veloiq_framework/cli/explorer.py`.

### Install framework from local source (development)
```bash
pip install -e backend/
```

## Architecture

### Framework (`veloiq_framework`)
- `create_veloiq_app(cfg)` — factory function that builds a FastAPI app with SQLAdmin, CORS, auth, module auto-loading, and extension discovery
- `VeloIQConfig` — dataclass-based config (reads env vars; override per-field for app-specific values)
- Module auto-loader: scans `app/modules/*/` for `models.py`, `api.py`, `custom_api.py`, `admin/admin_views.py`
- Built-in auth: `veloiq_framework.auth` — User/Role/Tenant models, JWT login, RBAC middleware, DB seeding
- Extension system: `veloiq_framework.extension.VeloIQExtension` + `veloiq_framework.extensions.discover_extensions()` — discovers installed extension packages via the `veloiq.extensions` Python entry point group at app startup

### UI package (`@juicemantics/veloiq-ui`)
- `DynamicList`, `DynamicShow`, `DynamicCreate`, `DynamicEdit` — schema-driven CRUD pages
- `generateResources(models, moduleName)` — builds Refine resource definitions from ModelDef arrays
- `authSystemModels` — static ModelDef definitions for User, Role, Tenant CRUD pages
- Auth providers: `authProvider`, `accessControlProvider`, `httpClient`

### Creating a new application
```bash
pip install veloiq-framework
veloiq new my-app
cd my-app
# Follow the generated README.md
```

## Extension package architecture

Extension packages are pip-installable Python packages that add modules, schemas, and licensing to any host app without modifying the host app's code.

### Key files
- `backend/veloiq_framework/extension.py` — `VeloIQExtension` base class (subclass this in your extension)
- `backend/veloiq_framework/extensions.py` — `discover_extensions()` scans the `veloiq.extensions` entry point group
- `backend/veloiq_framework/cli/new_extension.py` — `veloiq new-extension <name>` scaffolds a new extension package
- `backend/veloiq_framework/cli/add_licensing.py` — `veloiq add-licensing` scaffolds a license module into a host app

### Extension manifest contract
An extension declares itself via `pyproject.toml`:
```toml
[project.entry-points."veloiq.extensions"]
myext = "myext.manifest:ExtensionManifest"
```
And subclasses `VeloIQExtension`:
```python
from veloiq_framework import VeloIQExtension

class ExtensionManifest(VeloIQExtension):
    name = "myext"
    modules_package = "myext.modules"      # loaded like host app modules
    static_dir = "static"                   # served at /ext/myext/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cesarlugos1s/VeloIQ](https://github.com/cesarlugos1s/VeloIQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
