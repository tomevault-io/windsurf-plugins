---
trigger: always_on
description: Guidance for AI coding agents (OpenAI Codex, Cursor, OpenCode, Aider, etc.) working in this repository. Format follows the [agents.md](https://agents.md/) open standard. Mirrors `CLAUDE.md` — keep both in sync when editing.
---

# AGENTS.md

Guidance for AI coding agents (OpenAI Codex, Cursor, OpenCode, Aider, etc.) working in this repository. Format follows the [agents.md](https://agents.md/) open standard. Mirrors `CLAUDE.md` — keep both in sync when editing.

## Project overview

Desktop AI agent application:

- **Frontend** (`src/`) — React 19 + Vite 7 + Tailwind CSS 4 + Radix UI.
- **Backend** (`src-api/`) — Hono 4 + `@anthropic-ai/claude-agent-sdk` + `@modelcontextprotocol/sdk` + `@linear/sdk`. Hosted as a Node server in dev (port 5126) and as a packaged sidecar binary in production (port 2620).
- **Desktop shell** (`src-tauri/`) — Tauri 2 + Rust, with `tauri-plugin-sql` (SQLite), `tauri-plugin-shell`, `tauri-plugin-fs`.
- **Branding** — product name, identifiers, theme, and icons live per brand under `branding/<slug>/`. Active brand selected via `branding.json` at the repo root. Only `branding/default/` is checked in; custom brand folders are gitignored.

## Setup commands

```bash
pnpm install                  # install all workspace deps

pnpm dev:all                  # API server + Tauri desktop app
pnpm dev:api                  # API only (port 5126)
pnpm dev:web                  # Web frontend only (port 3420) — fastest HMR, no Tauri cache
pnpm dev:app                  # Tauri shell — predev:app runs brand:sync + check-rust + ensure-api-binary first
```

Branding switch: `pnpm brand:sync -- --brand=<slug>`. `predev:app` and `prebuild` run brand-sync automatically. Default is `default`.

Required toolchain: Node ≥20, pnpm ≥9. Rust ≥1.70 only for Tauri builds.

## Build and release

```bash
pnpm build                                # frontend (Vite)
pnpm build:api                            # API workspace
./scripts/build.sh mac-arm                # full desktop build (mac-arm | mac-intel | linux | windows)
./scripts/build.sh mac-arm --sign         # default notarized release build, no bundled CLIs
./scripts/build.sh mac-arm --with-cli --sign   # optional oversized build with bundled CLIs
pnpm release:new                          # bump + tag flow
pnpm release:publish                      # publish artifacts
```

## Test commands

```bash
pnpm test:fast                # Frontend + API unit/integration (everyday default)
pnpm test                     # Frontend Vitest only (vitest.config.ts)
pnpm test:api                 # API Vitest (src-api/vitest.config.ts)
pnpm test:e2e                 # Real server spawn (src-api/vitest.e2e.config.ts)
pnpm test:gate                # Eval gate tier (EVALS_TIER=gate)
pnpm vitest run path/to/file  # Run a single file
pnpm test -t 'pattern'        # Run a single test by title
```

`pnpm test:all` spawns Playwright + real-server processes. Don't run it casually — reserve it for pre-release sweeps.

Test layout:
- `src/__tests__/` — React Testing Library
- `src-api/test/unit/` — mocked unit tests
- `src-api/test/integration/` — Hono `app.request` integration
- `src-api/test/e2e/` — real server spawn

## Validation gate

Run before opening a PR:

```bash
pnpm validate    # brand:check + lint + typecheck:all + format:check + check:component-size
```

Components are capped at **350 lines** — `scripts/check-component-size.mjs` enforces this in CI.

After editing any `src/` file, format it before validating to avoid lint failures:

```bash
npx oxfmt <file>
```

Linting uses [oxlint](https://oxc.rs/) (Rust-based, ~50× faster than ESLint). Configs in `.oxlintrc.json` (frontend) and `src-api/.oxlintrc.json` (backend).

## Code style

- **TypeScript** strict (`tsconfig.json`, `src-api/tsconfig.json`). Target Node ≥20 / modern browsers.
- **Path alias** `@/*` is **scoped per workspace** — resolves to `src/*` from the frontend tsconfig and to `src-api/src/*` from `src-api/tsconfig.json`. Do not import across the boundary.
- **Imports** sorted by oxfmt's built-in `sortImports` (configured in `.oxfmtrc.jsonc`).
- **IDs** use `crypto.randomUUID()` — never `Date.now()` (collisions on rapid calls).
- **Module-level constants** — extract regex, config objects, and stable props (e.g. plugin configs) to module scope; inline objects break React memoization.

### Frontend conventions (`src/`)

- **Styling** — Tailwind CSS 4, Radix UI primitives, `cn()` helper for conditional classes.
- **i18n** — every user-visible string goes through `useLanguage()`. Update **all 6 locales** (`en`, `zh`, `es`, `fr`, `hi`, `pt`) in `src/config/locale/`.
- **Stale closures** — in `useCallback` with sparse deps, read current values from refs, not state captured at creation.
- **Functional setState** — use `setState(prev => …)` when reading current state in async callbacks; never close over state during streaming.
- **Effect cleanup** — every `fetch()` in `useEffect` must use `AbortController` aborted in cleanup. Required for React 19 StrictMode double-mount.
- **Effects vs user intent** — track manual interactions with a ref and skip auto-effects after the user has acted.
- **try/catch/finally** — never unconditionally overwrite error status in `finally`; use a flag to track whether `catch` ran.

### Backend conventions (`src-api/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bravew/Neumar](https://github.com/bravew/Neumar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
