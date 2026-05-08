---
trigger: always_on
description: Guidelines for agentic coding agents working in this repository.
---

# AGENTS.md

Guidelines for agentic coding agents working in this repository.

## Project Overview

Smart Home Planner is a Home Assistant add-on for planning, documenting, and visualizing smart home ecosystems.

**Tech Stack:**
- Python 3 HTTP server (serves UI + storage API)
- Node.js WebSocket worker (syncs with Home Assistant registries)
- Vanilla HTML/CSS/JS frontend (no build step)
- Cytoscape.js for device map visualization

## Build/Deploy Commands

```bash
# Deploy to Home Assistant testing instance (requires Samba mount)
sh sync-samba.sh

# Build Docker image locally
docker build -t smart-home-planner smart-home-planner/

# Install Node.js dependencies (for sync worker)
cd smart-home-planner && npm install --omit=dev

# Run Python server locally for development
SHP_WEB_ROOT=smart-home-planner/src SHP_DATA_FILE=./data.json python3 smart-home-planner/server.py
```

## Linting/Formatting Commands

No linters are currently configured, but these are recommended:

```bash
# Python linting and formatting
pip install black ruff
black --check smart-home-planner/
ruff check smart-home-planner/

# JavaScript linting
npx eslint smart-home-planner/src/js/

# CSS linting
npx stylelint "smart-home-planner/src/css/*.css"
```

## Language Convention

**All code, comments, documentation, UI text, variable names, function names, commit messages, and documentation must be written in English.** This applies to both backend and frontend code.

## Documentation Hygiene

- When you change code, verify whether `smart-home-planner/DOCS.md` must be updated to reflect the new behavior and update it when needed.

## Code Style Guidelines

### Python

- **Indentation:** 4 spaces (no tabs)
- **Naming:** `snake_case` for functions, variables, and file names; `PascalCase` for classes
- **Type hints:** Not used in this codebase
- **Imports:** Standard library first, then third-party (alphabetical within groups)
- **Strings:** Double quotes for string literals
- **Error handling:** Use specific exception types, return sensible defaults on failure
- **Thread safety:** Use `threading.Lock()` for shared state (see `server.py`)
- **Functions:** Keep functions focused and under ~50 lines when possible
- **Comments:** Only for non-obvious logic; prefer self-documenting code

Example:
```python
def _sanitize_device_id(value):
    normalized = FILENAME_SAFE_PATTERN.sub("_", str(value or "").strip())
    normalized = normalized.strip("._")
    if not normalized:
        raise ValueError("Missing or invalid device id")
    return normalized
```

### JavaScript

- **Indentation:** 4 spaces (no tabs)
- **Naming:** `camelCase` for variables and functions; `PascalCase` for constructor functions
- **Modules:** Use ES modules (`import`/`export`), not CommonJS
- **Imports:** Node builtins with `node:` prefix, then third-party
- **Variables:** Use `const` by default, `let` only when reassignment is needed; never use `var`
- **Functions:** Prefer arrow functions for callbacks; regular functions for top-level
- **Equality:** Always use strict equality (`===` and `!==`)
- **Async:** Use `async`/`await` instead of raw Promises when possible
- **Strings:** Double quotes for string literals
- **Error handling:** Throw `Error` objects with descriptive messages

Example:
```javascript
async function loadHaRegistry(url) {
    try {
        const response = await fetch(url, { cache: 'no-store' });
        if (!response.ok) {
            throw new Error(`Registry request failed: ${response.status}`);
        }
        return await response.json();
    } catch (error) {
        console.error(`Failed to load registry from ${url}:`, error);
        return [];
    }
}
```

### CSS

- **Indentation:** 4 spaces (no tabs)
- **Naming:** `kebab-case` for class names and IDs
- **Variables:** Use CSS custom properties defined in `:root` (see `common.css`)
- **Colors:** Use CSS variables from the dark theme palette
- **Organization:** One property per line, alphabetical or logical ordering
- **Selectors:** Prefer class selectors over ID selectors for reusability

Example:
```css
.dashboard-card {
    background: var(--card-bg);
    border-radius: 14px;
    padding: 1.5rem;
    box-shadow: var(--shadow-lg);
    border: 1px solid var(--border-color);
}
```

### HTML

- **Indentation:** 4 spaces (no tabs)
- **Semantic elements:** Use `<header>`, `<main>`, `<nav>`, `<section>`, `<article>`
- **Accessibility:** Include `aria-*` attributes and `role` where appropriate
- **Naming:** `kebab-case` for IDs and classes
- **Quotes:** Double quotes for attribute values

## Project Structure

```
smart-home-planner/
├── config.yaml          # Home Assistant add-on configuration
├── Dockerfile           # Container build instructions
├── run.sh               # Entry point script
├── server.py            # Python HTTP server (UI + API)
├── registry-sync.js     # Node.js WebSocket sync worker
├── ha-device-update.js  # Node.js script for updating HA devices
├── package.json         # Node.js dependencies
├── src/                 # Frontend files
│   ├── index.html       # Main dashboard
│   ├── devices.html     # Device list page
│   ├── device-add.html  # Add device form
│   ├── device-edit.html # Edit device form
│   ├── settings.html    # Settings page

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [smarthomecompared/smart-home-planner](https://github.com/smarthomecompared/smart-home-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
