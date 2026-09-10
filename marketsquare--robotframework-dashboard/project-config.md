---
trigger: always_on
description: This file gives AI agents and contributors the context needed to work effectively in this codebase.
---

# robotframework-dashboard — Copilot Instructions

This file gives AI agents and contributors the context needed to work effectively in this codebase.

## How to Use the Skills Files

Before starting work on any non-trivial task, read the relevant skill file from `.github/skills/`. Each file contains deep domain knowledge that avoids re-exploring the codebase from scratch. Use the table in the **Skills** section below to pick the right one(s). Read the skill file with a file-read tool before making any changes.

---

## Commands

Use the project scripts — do NOT invoke the underlying tools directly (the scripts set coverage paths, artifact dirs, and parallelism). `.bat` for Windows, `.sh` for Linux/macOS.

| Task | Windows | Linux / macOS |
|---|---|---|
| JS unit tests | `scripts\javascript-tests.bat` | `bash scripts/javascript-tests.sh` |
| Python unit tests | `scripts\python-tests.bat` | `bash scripts/python-tests.sh` |
| Robot acceptance tests | `scripts\robot-tests.bat` | `bash scripts/robot-tests.sh` |
| Generate dashboard for testing | `python -m robotframework_dashboard.main -n robot_dashboard -f tests` | same |
| Docs build | `npm run docs:build` | `npm run docs:build` |
| Docs dev server | `npm run docs:dev` | `npm run docs:dev` |

**Generate dashboard for testing** runs the package directly (no install) against the `tests/` output.xml fixtures, producing `robot_dashboard.html`. Use this to validate any JS/CSS/template/Python pipeline change — open the HTML to confirm rendering, layout, and click handlers. A clean import/syntax check is not sufficient; bundled-output bugs only surface here.

---

## Project Purpose

`robotframework-dashboard` is a Python CLI tool that reads Robot Framework `output.xml` execution results, stores them in a SQLite database, and generates a fully self-contained HTML dashboard with interactive charts, tables, and filters. No web server is required to view the output — a single `.html` file contains all data, JS, and CSS.

---

## Core Pipeline: Python CLI → HTML Template → JavaScript

The entire system is this three-stage pipeline:

```
1. PYTHON CLI
   output.xml files
       └─► OutputProcessor (robot.api ResultVisitor)
               └─► SQLite database (runs / suites / tests / keywords tables)

2. HTML TEMPLATE
   database.get_data()
       └─► DashboardGenerator
               ├─► DependencyProcessor: merges all JS modules (topological sort) → inline <script>
               ├─► DependencyProcessor: merges all CSS files → inline <style>
               ├─► CDN or offline dependency tags
               ├─► Data encoded as: JSON → zlib compress → base64 → string literal in HTML
               └─► templates/dashboard.html (string placeholder replacement) → robot_dashboard.html

3. JAVASCRIPT (runs in the browser)
   js/variables/data.js decodes the embedded base64 data back to JS arrays
       └─► Chart.js charts, DataTables, filters, layout — all from local data, zero server calls
```

The output is a **single `.html` file** that is entirely self-contained. All Robot Framework data is embedded as compressed strings; all JS and CSS is inlined.

---

## Entry Points

| File | Role |
|---|---|
| `robotframework_dashboard/main.py` | CLI entry point (`robotdashboard` command) |
| `robotframework_dashboard/robotdashboard.py` | `RobotDashboard` class — orchestrates all 5 pipeline steps |
| `robotframework_dashboard/arguments.py` | `ArgumentParser` wrapping `argparse` |
| `robotframework_dashboard/processors.py` | `OutputProcessor` + 4 `ResultVisitor` subclasses |
| `robotframework_dashboard/database.py` | Built-in SQLite implementation |
| `robotframework_dashboard/abstractdb.py` | `AbstractDatabaseProcessor` ABC (custom DB backends) |
| `robotframework_dashboard/queries.py` | All SQL strings as module-level constants |
| `robotframework_dashboard/dashboard.py` | `DashboardGenerator` — template rendering |
| `robotframework_dashboard/dependencies.py` | `DependencyProcessor` — JS/CSS inlining and CDN switching |
| `robotframework_dashboard/server.py` | Optional FastAPI server (`--server` flag) |

---

## JavaScript and CSS

All frontend source lives under `robotframework_dashboard/js/` and `robotframework_dashboard/css/`. **There is no Node.js bundler (no webpack, Vite, or Rollup) for the dashboard.** Bundling is done in Python by `DependencyProcessor` at HTML generation time.

Key JS directories:

| Path | Contents |
|---|---|
| `js/variables/` | Global state, data decoding, settings, graph registry |
| `js/graph_creation/` | Chart.js setup per tab (overview, run, suite, test, keyword, compare, tables) |
| `js/graph_data/` | Data transformation modules that feed Chart.js |
| `js/main.js` | Startup entry — imports and calls all setup functions |
| `js/admin_page/` | Separate JS bundle for the server's `/admin` page only |

See `.github/skills/js-bundling.md` for details on how JS modules are resolved, ordered, and embedded.

---

## HTML Templates

Templates live in `robotframework_dashboard/templates/`. They use simple string placeholder tokens (not Jinja2):

- `templates/dashboard.html` → generates `robot_dashboard.html`
- `templates/admin.html` → generates the server's `/admin` page


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarketSquare/robotframework-dashboard](https://github.com/MarketSquare/robotframework-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
