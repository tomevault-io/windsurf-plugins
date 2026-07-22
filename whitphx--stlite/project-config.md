---
trigger: always_on
description: **Stlite** is a serverless Streamlit running in browsers via Pyodide (Python-to-WebAssembly). No backend server needed.
---

# CLAUDE.md - Stlite AI Assistant Guide

**Stlite** is a serverless Streamlit running in browsers via Pyodide (Python-to-WebAssembly). No backend server needed.

- **Stack**: React + TypeScript + Vite + Pyodide(Python)
- **Monorepo**: Yarn workspaces, Makefile-orchestrated builds
- **Testing**: Vitest + Playwright
- **Streamlit fork**: Git submodule at `streamlit/` (forked customized branch named `stlite-<version>`)
  - **Branch naming and force-push policy**: While an `stlite-<version>` branch hasn't been released yet, force-pushing it on `whitphx/streamlit` is acceptable — we treat it as a working branch. Once the version is released, the branch is frozen; the next round of fixes goes onto a new branch named `stlite-<version>-2` (and so on) rather than rewriting history on the released branch.

## Repository Structure

```
packages/
  common/          # Shared TS types (base package, no deps)
  kernel/          # Core Pyodide runtime & worker management
    py/stlite-lib/ # Python: custom Streamlit server (replaces streamlit.web)
    py/streamlit/  # Compiled Streamlit wheel for Pyodide
  react/           # React bindings (StliteApp, StliteAppWithToast)
  browser/         # Browser-mountable API (mount(), <streamlit-app>)
  desktop/         # Electron desktop app
  sharing/         # App sharing service
  sharing-editor/  # Editor app for the sharing service (Monaco editor)
  sharing-common/  # Shared code for sharing (protobuf)
  tooling/         # Build utilities
streamlit/         # Git submodule: whitphx's Streamlit fork
docs/              # Documentation site (Astro Starlight), see below
.make/             # Sentinel files for incremental builds
```

**Dependency graph**: common -> kernel -> react -> browser / desktop / sharing

## Development Setup & Building

See `CONTRIBUTING.md` for environment setup, dev workflows per package, and building.

**Key points for AI assistants**:

- **Always use Makefile targets** (`make browser`, `make kernel`, etc.) - they handle dependency ordering via sentinel files in `.make/`.
- **Do NOT run `cd packages/X && yarn build` directly** - dependencies may not be built.
- `NODE_OPTIONS="--max-old-space-size=6144"` is set in Makefile to prevent heap errors.

## Testing

```bash
# Unit tests (Vitest) - per package
cd packages/<pkg> && yarn test
cd packages/<pkg> && yarn test:watch
make kernel-test   # With proper build deps

# E2E tests (Playwright)
cd packages/browser/e2e-tests
yarn install && yarn install:browsers && yarn test

# Python tests (pytest)
cd packages/kernel/py/stlite-lib && uv run pytest
```

- Vitest uses `jsdom` environment (not happy-dom, due to iframe issues)
- Prefer `make kernel-test` for kernel tests (it builds required wheels). Run `make stlite-lib-wheel streamlit-wheel` manually only if you call `cd packages/kernel && yarn test` directly.
- Test file naming: `*.spec.ts` for pure unit tests; `*.test.ts` for heavier integration tests that exec subprocesses, load Pyodide, or otherwise reach beyond a single module (e.g. `kernel.test.ts`, `worker-runtime.test.ts`, `parity.test.ts`). Vitest picks up both.

## Code Quality

```bash
# TypeScript
cd packages/<pkg>
yarn check:lint # or yarn fix:lint   (oxlint)
yarn check:format # or yarn fix:format (oxfmt)

# Python (stlite-lib)
cd packages/kernel/py/stlite-lib
uv run ruff check . && uv run ruff format . && uv run pyright
```

- Lint: oxlint (`.oxlintrc.json` at repo root), Format: oxfmt (`.oxfmtrc.json`, 2 spaces, semicolons, printWidth 80)
- Unused vars/args with `_` prefix are allowed repo-wide (`argsIgnorePattern: "^_"` etc.)
- Git hooks (Husky): pre-commit runs lint-staged (oxfmt on all formatter-supported files, oxlint --fix on JS/TS)
- Commits: conventional format `type(scope): summary`

## Changeset Management

Each PR that introduces user-facing changes (new features, bug fixes, breaking changes) MUST include a Changeset file created via `yarn changeset`. This ensures proper versioning and changelog generation.

```bash
yarn changeset
```

**For AI assistants**: After implementing any meaningful user-facing change (new feature, bug fix, dependency update like a Streamlit rebase, breaking change), ask the user whether a changeset fragment should be added so it isn't forgotten before the PR is opened.

## Architecture

**Worker-based**: Python runs in Web Workers (Dedicated or SharedWorker) via Pyodide. Main thread handles React UI, communicates via `postMessage`.

**Streamlit integration**: Stlite uses `streamlit.runtime` but replaces `streamlit.web` (Tornado) with `stlite-lib` (custom server at `packages/kernel/py/stlite-lib/stlite_lib/`). Key files: `server/Server.py`, `codemod.py` (AST transforms).

**File systems**: MEMFS (default, ephemeral), IDBFS (persistent via IndexedDB), NODEFS (desktop only).

## Documentation, Demos & E2E Tests

### Documentation Site (`docs/`)

Astro Starlight site with pages for `@stlite/browser`, `@stlite/react`, and `@stlite/desktop`. Uses a **single source of truth** approach: demo files and package versions are imported directly from the workspace packages, not duplicated.

### Demos as Shared Fixtures


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whitphx/stlite](https://github.com/whitphx/stlite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
