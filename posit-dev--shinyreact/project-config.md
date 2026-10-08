---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

`shinyreact` is a monorepo providing React UI infrastructure for Shiny (Python and R). It provides zero UI components — it is the bridge between a Shiny server that contains only reactive computation and a React client the app author owns.

The repo ships one first-class pattern: the **`ui.tsx` pattern** — `set_react_page()` (Python Express) / `page_react()` (Python Core, R) bootstraps a React client whose entry conventionally lives in `ui.tsx` (compiled to `www/ui.js`, discovered automatically); the client and server communicate through the `useShinyInput` / `useShinyOutputValue` hook family. See `DESIGN.md` for background.

## Terminology

**`ui.tsx`** is the canonical name of the pattern. **Never write "SPA", "Single Page App", "Single-Page Application", "traditional pattern", `client-ui`, or `ui-object`** in new content (docs, comments, commit messages, PR/issue text).

- **`ui.tsx` pattern** — UI defined in a client-side codebase whose entry is conventionally `ui.tsx` (or `ui.jsx`, or `ui.js` for no-build); bootstrapped from the app file via `set_react_page()` / `page_react()` (or `page_react_html()` for apps that own a full HTML document). `ui.tsx` is the *idiomatic* canonical name — examples use the same role at different tiers: `www/ui.js` (no-build) or `src/ui.jsx` (Vite + JSX, built to `www/ui.js`). Treat `ui.tsx` as a *role label* for the React entry, not a strict filename requirement.
- The phrase "traditional Shiny" is fine when it refers to vanilla Shiny (no shinyreact involved).
- The repo formerly also shipped an **`app.py` pattern** (server-side JSON-spec rendering via `Node` / `render_react` / `page_react`). It was removed in #168; see the tracking comment on #167 for git-history pointers if you encounter stale references.

## Repo structure

```
pkg-js/                     # TypeScript/React Vite IIFE bundle
  src/                      # index.ts, global.ts, shiny-output.tsx, shiny.d.ts, shinyreact.css
  dist/                     # Built assets (committed to repo)
  src/shiny-react/          # Vendored @posit/shiny-react source (hooks, registries)
pkg-py/                     # Python package
  src/shinyreact/           # React-bridge package
    www/                    # Bundled JS
  tests/                    # pytest tests (+ tests/playwright/ e2e)
pkg-r/                      # R package — mirrors the Python API in R
  R/                         # render.R, page.R, message.R, bookmark.R, dep.R, input-handler.R
  inst/lib/shiny/            # Bundled JS (R counterpart of pkg-py www/)
  tests/testthat/            # testthat tests
examples/                   # ui.tsx pattern examples (01-hello … 10-bookmarking)
FEATURES.md                 # behavior tree for all three packages — see below
docs/                       # posit-conf-2026-goals.md, historical plans/specs
decisions/                  # Architecture decision records
pyproject.toml              # Root-level, hatchling backend
Makefile                    # All build/check/format commands
```

## Commands

```bash
# Initial setup
uv sync --all-extras --all-groups   # Python env
make js-setup                        # JS deps (cd pkg-js && npm install)
pre-commit install                   # Pre-commit hooks

# Build
make js-build                        # Build JS bundle (pkg-js/dist/)
make update-dist                     # Build JS + copy to pkg-py/www/ and pkg-r/inst/lib/shiny/
make update-examples                 # Copy examples/*/app.R + www/ to pkg-r/inst/examples-shiny/

# Python checks (run all before committing)
make py-check                        # format check + type check + tests
make py-check-tests                  # pytest only
make py-check-types                  # pyright only
make py-format                       # ruff fix + format
make py-check-tox                    # full matrix: Python 3.10–3.14

# JS checks
make js-lint                         # tsc --noEmit
make js-format                       # prettier --write (js-format-check in CI)
make js-build-watch                  # watch mode

# R checks
make r-check                         # format + tests + R CMD Check
make r-format                        # air format

# Run a single Python test
uv run pytest pkg-py/tests/test_page.py::test_name

# Update test snapshots
make py-update-snaps

# Documentation site (renders into gitignored pkg-py/docs/_site/: Quarto landing
# page + quartodoc Python API at py/, pkgdown at r/, TypeDoc at js/)
make docs                            # py-docs + r-docs-render + js-docs
make docs-preview                    # serve pkg-py/docs/_site on :8080
```

Run `make help` to see all targets.

## Architecture

### JS bundle

The JS output (`pkg-js/dist/shinyreact.js`) is a self-contained IIFE that bundles React 19 and vendored `@posit/shiny-react`, and installs the public API at `window.shinyreact`.

**Global API exposed at `window.shinyreact`:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [posit-dev/shinyreact](https://github.com/posit-dev/shinyreact) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
