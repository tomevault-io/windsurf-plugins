---
trigger: always_on
description: MCP App template: React widgets + Python MCP server. Build once, run on Claude, ChatGPT, VS Code, Goose, and other MCP hosts.
---

# AGENTS.md

MCP App template: React widgets + Python MCP server. Build once, run on Claude, ChatGPT, VS Code, Goose, and other MCP hosts.

## Quick Start (Follow These Steps)

### 1. Setup and Verify
```bash
./setup.sh   # Installs deps, builds, runs tests — must pass before continuing
```

### 2. Understand the Framework
Read in this order:
- `README.md` — What MCP Apps are, why this template exists
- `docs/widget-development.md` — Hook APIs and widget patterns

### 3. Review Examples (By Complexity)

Start with simpler examples and progress to advanced ones as needed:

| Complexity | Widget | Description | Key Concepts |
|------------|--------|-------------|--------------|
| **Beginner** | `list` | Simple list display | Basic props, theme support |
| **Beginner** | `qr` | QR code generator | Single input, canvas rendering |
| **Intermediate** | `carousel` | Image slideshow | State management, navigation |
| **Intermediate** | `gallery` | Image grid | Grid layout, responsive design |
| **Intermediate** | `dashboard` | Stats cards | Multiple data types, layout |
| **Advanced** | `scenario-modeler` | Interactive charts | Chart.js, complex data |
| **Advanced** | `solar-system` | 3D visualization | Three.js, animation loops |
| **Advanced** | `map` | Interactive maps | External APIs (Leaflet), geocoding |

**Recommended starting point**: Examine the `carousel` widget:
- **Frontend**: `src/carousel/index.tsx` (entry) + `src/carousel/App.tsx` (component)
- **Server**: `server/widgets/carousel.py` — exports `WIDGET`, `INPUT_MODEL`, `handle()`

### 4. Create Your App
```bash
./create_new_app.sh --name my_app                    # Start fresh
./create_new_app.sh --name my_app --keep carousel    # Keep carousel as reference
```
Verify tests still pass after running the script.

### 5. Development Loop
For each widget you build:
```bash
# 1. Create src/my-widget/index.tsx + src/my-widget/App.tsx
# 2. Create server/widgets/my_widget.py (auto-discovered)
# 3. Build and test:
pnpm run build && pnpm run test
# 4. Visual verification:
pnpm run ui-test --tool show_my_widget
# 5. Check screenshot: /tmp/ui-test/screenshot.png
# 6. Iterate until tests pass and widget renders correctly
```

### 6. Final Verification
```bash
pnpm run test:browser   # Slower but catches rendering issues in real browser
```

Your app is production-ready.

---

## Commands Reference

| Command | Purpose |
|---------|---------|
| `./setup.sh` | First-time setup (installs deps, builds, tests) |
| `pnpm run build` | Build all widgets (REQUIRED before server) |
| `pnpm run test` | Server + UI tests (run after every change) |
| `pnpm run test:all` | All tests including browser |
| `pnpm run test:browser` | Browser compliance tests only |
| `pnpm run server` | Start MCP server at localhost:8000 |
| `pnpm run ui-test --tool <name>` | Render tool, save screenshot to `/tmp/ui-test/` |
| `pnpm run ui-test --tool <name> --theme dark` | Test tool in dark mode |

---

## Boundaries

### Always
- Run `pnpm run build` before `pnpm run server` or `pnpm run test`
- Run `pnpm run test` after every code change
- Add `extra='forbid'` and default values to all Pydantic Input models
- Support both light and dark themes in widgets
- Check test grade reports after running tests:
  - `server/tests/mcp_best_practices_report.txt`
  - `server/tests/chatgpt_app_guidelines_report.txt`
  - `server/tests/output_quality_report.txt`
  - `server/tests/agentic_ux_patterns_report.txt`
  - `server/tests/protocol_compliance_report.txt`
  - `server/tests/app_submission_report.txt`

### Ask First
- Adding new npm dependencies (especially heavy ones like `three`, `chart.js`)
- Changing the MCP server's transport or security settings
- Modifying shared hooks behavior (`useWidgetProps`, `useTheme`)

### Never
- Modify `internal/apptester/` — Testing infrastructure, not an example
- Modify `internal/sandbox-proxy/` — Testing infrastructure, not an example
- Modify `src/use-*.ts` hooks — Shared infrastructure used by all widgets
- Modify `tests/browser/apptester-e2e.spec.ts` — Core test infrastructure
- Create Input models without `extra='forbid'` (causes test failures)
- Skip running tests after changes

---

## Tech Stack

| Layer | Technology |
|-------|------------|
| Widgets | React 18, TypeScript, Tailwind CSS |
| Server | Python 3.11+, FastMCP, Pydantic |
| Protocol | MCP Apps (SEP-1865) |
| Testing | Vitest (UI), pytest (server), Playwright (browser) |

---

## File Structure

| Path | Purpose |
|------|---------|
| `src/{widget}/index.tsx` | Widget entry point (targets `{widget}-root` element) |
| `src/{widget}/App.tsx` | Widget React component (optional but recommended) |
| `src/use-*.ts` | Shared hooks — DO NOT MODIFY |
| `server/widgets/{widget}.py` | Widget server module (auto-discovered) |
| `server/widgets/_base.py` | Base classes and helpers |
| `server/main.py` | MCP server setup (rarely needs editing) |
| `internal/` | Test infrastructure — DO NOT MODIFY |
| `tests/*.test.ts` | UI unit tests (Vitest) |
| `tests/browser/*.spec.ts` | Browser compliance tests (Playwright) |
| `server/tests/test_*.py` | Server tests and grading (pytest) |

---

## Adding a Widget (Quick Reference)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sebderhy/mcp-app-template](https://github.com/sebderhy/mcp-app-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
