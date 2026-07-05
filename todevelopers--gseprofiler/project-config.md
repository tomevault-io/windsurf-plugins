---
trigger: always_on
description: GTK4 desktop application for **managing, debugging, and profiling GNOME Shell extensions**.
---

# CLAUDE.md — gse-profiler

## Project Overview

GTK4 desktop application for **managing, debugging, and profiling GNOME Shell extensions**.
Targets GNOME Shell extension developers. Internally installs a bridge GJS extension that acts as a bridge into the `gnome-shell` process.

**Target platform:** GNOME 48+ (Wayland only)
**Languages:** Python 3.11+ (app), GJS / ES6 (bridge extension + developer API)

---

## Architecture

```
GTK4 App (Python/PyGObject)
         │
         ├─ D-Bus ──────────────► org.gnome.Shell.Extensions  (list / enable / disable)
         │
         └─ Unix Socket (JSON) ──► Bridge Extension (GJS)
                                         │
                                         ├── Target extension   (monkey-patch, inspect)
                                         ├── Core gnome-shell   (optional)
                                         └── opt-in API bridge  (DevToolsClient)
```

Communication split:

- **D-Bus** — standard GNOME Shell APIs (extensions list, enable/disable)  
- **Unix socket** — all custom high-frequency communication with the bridge

Socket path: `$XDG_RUNTIME_DIR/gse-profiler.sock`  
Protocol: newline-delimited JSON messages

---

## Directory Layout

```
gse-profiler/
├── app/                        # GTK4 Python application
│   ├── main.py
│   ├── ui/                     # UI view modules
│   │   ├── extension_manager.py
│   │   ├── log_viewer.py
│   │   ├── profiler_view.py
│   │   └── inspector_view.py
│   ├── core/                   # Core logic
│   │   ├── dbus_client.py
│   │   ├── socket_server.py
│   │   ├── git_manager.py
│   │   └── journal_reader.py
│   └── data/ui/                # Glade .ui files (if needed)
├── bridge-extension/        # GJS GNOME Shell extension
│   ├── extension.js
│   ├── profiler.js
│   ├── inspector.js
│   ├── socket_client.js
│   └── metadata.json
├── api/
│   └── devtools-api.js         # opt-in developer API
└── tests/                      # pytest unit tests
```

---

## Bridge Extension

- **UUID:** `gse-profiler-bridge@todevelopers`
- **Install path:** `~/.local/share/gnome-shell/extensions/gse-profiler-bridge@todevelopers/`
- Auto-installed by the app on first launch
- Shows a minimal status indicator in the GNOME panel (no menu needed in V1)
- After installation, the app triggers a shell restart via D-Bus (see below)

### Shell Restart Logic

Performed via a single session-bus D-Bus call:
`org.gnome.SessionManager.Logout(1)` (no-confirm logout). Works
identically inside and outside Flatpak.

---

## Coding Conventions

### Python (`app/`)

- Python 3.11+, PEP 8, type hints throughout
- Use GObject property system (`GObject.Property`) where appropriate
- D-Bus calls: use `Gio.DBusProxy` async variants — never block the main loop
- Unix socket I/O: async via `GLib.IOChannel` or `asyncio` with GLib event loop integration
- No hardcoded paths — use `GLib.get_user_data_dir()`, `GLib.get_runtime_dir()`, etc.

### GJS (`bridge-extension/`, `api/`)

- ES6 module syntax (`import` / `export`), strict mode (`'use strict'`)
- Use GNOME GJS bindings (`imports.gi.*` or `gi://` depending on GNOME version)
- Always disconnect signals in `disable()` — no memory leaks
- Never use `org.gnome.Shell.Eval` — all introspection goes through the socket

### General

- All user-facing strings and identifiers in English
- No `console.log` left in production GJS code — use `log()` / `logError()`
- Profile event JSON schema: `{ type, extensionUuid, function, start, end, depth }`

---

## D-Bus Interfaces Used

| Interface                    | Purpose                            |
| ---------------------------- | ---------------------------------- |
| `org.gnome.Shell`            | List extensions, enable/disable (via the `org.gnome.Shell.Extensions` interface) |
| `org.gnome.SessionManager`   | Logout after bridge install/remove |
| `org.freedesktop.DBus`       | Introspection                      |

---

## Headless Smoke Testing on Windows (WSL)

The repo lives on Windows but the app targets GNOME/Linux. Full UI testing
needs a real GNOME 48+ box (D-Bus, Wayland, gnome-shell). For everything
short of that — syntax, imports, widget construction, draw functions —
use WSL. PyGObject 3.48+, GTK4, and libadwaita-1 are typically already
installed on a recent Ubuntu WSL.

> **These checks run automatically via the Claude Code Stop hook**
> (`.claude/run-tests.ps1`): `ruff check app/`, `eslint bridge-extension/ api/`,
> syntax check, WSL headless tests, and `pytest`. If any check fails, Claude
> is blocked from finishing and must fix the errors first. Manual runs below
> are for debugging only.

**1. Syntax check (Windows Python is fine here, no `gi` needed):**

```bash
py -c "
import ast
for p in [
    'C:/GitHubRepos/gse-profiler/app/ui/profiler_view.py',
    # …add the files you touched
]:
    with open(p, encoding='utf-8') as f: ast.parse(f.read())
    print('OK:', p)
"
```

**2. Import check — every module under WSL** (catches missing names, bad
relative imports, signal-type mismatches at class-construction time):

```bash
wsl -- bash -c "cd /mnt/c/GitHubRepos/gse-profiler && python3 -c '
import sys, os; sys.path.insert(0, os.getcwd())
from app.ui.profiler_view import ProfilerView

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [todevelopers/gseprofiler](https://github.com/todevelopers/gseprofiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
