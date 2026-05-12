---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Read First

Before making changes, read these repo docs:

- `docs/product.md` for product principles and UX direction
- `docs/engineering.md` for stack, file structure, and engineering boundaries

`AGENTS.md` is the source of truth for coding conventions and agent workflow rules.

## Commands

Build the web app:

```bash
bun run build:web
```

This command:

1. Removes the existing `_site` directory
2. Copies static files from `static/` to `_site/`
3. Runs the Rettangoli CLI to build the frontend bundle to `_site/public/main.js`

Notes:

- `bun run build` may not exist in this repo; use `build:web` for validation.
- Do not run `bun run build:web` after each change. The user is expected to be running a watch-mode session during active development.
- Before pushing, run lint/format checks (the push hook also enforces this).

Run tests:

```bash
bun run test:smoke
bun run test:integration
bun run test:convergence
bun run test:collab-adapters
bun run test:puty
```

Notes:

- `bun run test:puty` runs the YAML-based Puty storage suite in `tests/puty/`.
- Run a single Puty scenario with `bunx vitest run tests/puty/<file>.spec.yaml`.
- The Puty suite is the preferred place for SQLite-backed storage assertions:
  input commands live in YAML `in`, expected committed rows live in YAML `out`.
- The shared Puty helper for storage scenarios is `tests/puty/insiemeStorageScenario.js`.
- The `scripts/test-*.js` files remain the home for non-Puty runtime,
  integration, convergence, and command-contract coverage.

## Architecture

This project uses a custom frontend framework based on 3 component files: view, store, handlers:

Read the links from the following files to familiarize with the code before starting to write any code.

- [Overview](https://raw.githubusercontent.com/yuusoft-org/rettangoli/refs/heads/main/packages/rettangoli-fe/docs/overview.md)
- [View](https://raw.githubusercontent.com/yuusoft-org/rettangoli/refs/heads/main/packages/rettangoli-fe/docs/view.md)
- [State Management](https://raw.githubusercontent.com/yuusoft-org/rettangoli/refs/heads/main/packages/rettangoli-fe/docs/store.md)
- [Handlers](https://raw.githubusercontent.com/yuusoft-org/rettangoli/refs/heads/main/packages/rettangoli-fe/docs/handlers.md)

If you need deeper or broader Rettangoli framework reference material, use
`https://rettangoli.dev/llms.txt` as the framework reference index.

## JavaScript Style

- Prefer direct property access or nullish coalescing (`??`) for defaults.
- Use `??` for default values instead of `||` when setting state/object fields.
- Avoid defensive `typeof x === "string" ? x : ""` patterns unless runtime type narrowing is truly required.
- Prefer `undefined` over `null`.
- Avoid explicit `= null` initialization; use `let value;` when a later assignment is expected.
- If state already guarantees a value, use it directly instead of re-normalizing with fallback checks.
- Do not build objects with conditional object-spread patterns such as `...(condition ? { field } : {})` or nested spread-based payload builders. Build a local object and assign fields explicitly.
- In Immer-backed store actions, prefer direct mutation of nested state fields over recreating nested objects with spread. Write `state.dialog.open = false`, not `state.dialog = { ...state.dialog, open: false }`.

## Layering

- Keep page/component handlers simple and orchestration-focused.
- Keep page `*.store.js` and `*.handlers.js` as the composition root for the
  page. They should stay easy to find and should wire components and shared
  helpers together explicitly.
- Push domain logic, validation, and async complexity into services.
- Route-level async setup/loading should be handled in app-level orchestration, not repeated in page handlers.
- Prefer single-purpose store actions (`setCurrentProject`, etc.) over multiple related setter calls.
- Do not call browser globals like `document` or `window` directly from page handlers for cross-cutting side effects.
- If a handler needs browser-side behavior such as blurring the active element, global focus changes, history changes, or global listeners, route that through `deps/services` or `deps/clients`.
- Keep low-level DOM-heavy behavior in `src/primitives/` or in components that own that DOM surface directly.
- `src/internal/ui/` is the shared home for app-owned page/store/handler orchestration.
- `src/internal/project/` is reserved for pure project semantics only.
- Small pure app-owned helpers that are neither project semantics nor UI orchestration belong in `src/internal/`.
- `src/deps/features/` and `src/deps/infra/` are legacy folders. Do not add new code there.

## Detail Panel Pattern

- Use `rvn-detail-view` for read-only right panels (instead of read-only forms).
- Build read-only data in store as `detailFields` (types: `text`, `description`, `slot`).
- Prefer `text` over `text-inline` unless explicitly required.
- Keep custom interactive UI (preview button, lists, actions) as slots inside `rvn-detail-view`.
- Place panel title/header outside `rvn-detail-view` when needed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RouteVN/routevn-creator-client](https://github.com/RouteVN/routevn-creator-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
