---
trigger: always_on
description: npm run build      # typecheck (tsc --noEmit) → esbuild bundle + zip
---

# AGENTS.md

## Build commands

```
npm run build      # typecheck (tsc --noEmit) → esbuild bundle + zip
npm run dev        # typecheck → esbuild watch + serve on port 3000
```

Build order matters: typecheck must pass before esbuild runs. Output is `dist/main.js` → automatically zipped to `dist.zip` (the plugin artifact).

## Architecture

Acode plugin (Android WebView) that launches OpenCode as a background HTTP server inside Acode's built-in Alpine Linux terminal, then embeds its web UI in an iframe.

**Entrypoint:** `src/main.ts` → `AcodePlugin` class.

**Directory layout** - feature-based

**State machine** (`src/types.ts` — `AppState` enum): `Idle → CheckingInstall → Installing → CheckingServer → StartingServer → Ready`. Error can be entered from any state. UI is purely reactive via `onStateChange` listener.

## Hard constraints (non-negotiable)

- **Fixed port 4096, loopback only.** `opencode serve --port 4096 --hostname 127.0.0.1`. Never bind to `0.0.0.0` without adding auth.
- **Executor.execute is blocking AND session-reaping.** The terminal module resolves only after the command exits, and it treats the command as finished the moment its stdout pipe hits EOF — then tears down the shell session, reaping any backgrounded child. For persistent processes, use `Executor.BackgroundExecutor` (which persists across session teardown, returns a UUID via `start()`, and is stopped via `stop(uuid)`). `disown` is a bash-ism not available in BusyBox `ash` (Acode's Alpine shell).

## Code conventions

- **SOC** SEPERATION OF CONCERN (u MUST follow that rule)
- **No magic numbers/strings.** All constants live in `src/config/`. Import them — never inline literals. Consumers import from the specific sub-module (e.g. `./config/server`), not the barrel, except tests.
- **ESLint rules** (enforced when eslint is installed): `max-depth: 3`, `max-lines-per-function: 40` (warn), `no-magic-numbers` (warn, except 0,1,-1), `prefer-const`, `no-var`.
- **Prettier** (when installed): single quotes, trailing commas, 100 char width, 2-space tabs.
- **Untyped Acode modules** use `acode.require('...') as any` — this is the established pattern (acode-plugin-types doesn't cover all runtime modules).
- **State handling** uses the `AppState` enum and `transition()` instead of nested conditionals. Never add a state check without adding it to the enum.
- **Imports:** `config/` is a leaf package of pure constants — sub-modules import nothing outside `config/` (cross-config imports like `health.ts` ← `server.ts` are allowed; no imports from `../opencode`, `../ui`, etc.).

## Gotchas

- `eslint` and `prettier` packages are **not installed** (configs exist but are inert). Install them before running lint/format.
- Tests use Vitest with jsdom (`npm test`). Test files live under `test/`, mirroring the `src/` tree (e.g. `test/opencode/server.test.ts`), and import from `../src/...` / `../../src/...`. `vitest.config.ts` `include` is `test/**/*.test.ts`; tsconfig excludes `test/**/*`.
- `plugin.json` is configured for publishing: `id` `com.victozee26.opencode`, `name` `OpenCode`, `version` `0.1.0`, `main` `dist/main.js`, author set from git identity. Bump `version` + `changelogs.md` for each release.
- `html-tag-js` is listed as a dependency but currently unused — all DOM is vanilla `document.createElement`.
- Health-check uses `cordova.plugin.http` (Cordova Advanced HTTP) only — it runs on the native network stack so WebView CORS does NOT apply and a loopback probe to `127.0.0.1:4096` actually resolves (a plain `fetch` to loopback hangs forever in this WebView). There is NO `fetch` fallback: `isServerUp()` returns `false` immediately when `cordova.plugin.http` is absent, so tests stub `window.cordova.plugin.http` rather than relying on a browser `fetch`. Do not reintroduce a `fetch` probe. See `src/opencode/AGENTS.md`.

# DOX framework

- DOX is highly performant AGENTS.md hierarchy installed here
- Agent must follow DOX instructions across any edits

## Core Contract

- AGENTS.md files are binding work contracts for their subtrees
- Work products, source materials, instructions, records, assets, and durable docs must stay understandable from the nearest applicable AGENTS.md plus every parent AGENTS.md above it

## Read Before Editing

1. Read the root AGENTS.md
2. Identify every file or folder you expect to touch
3. Walk from the repository root to each target path
4. Read every AGENTS.md found along each route
5. If a parent AGENTS.md lists a child AGENTS.md whose scope contains the path, read that child and continue from there
6. Use the nearest AGENTS.md as the local contract and parent docs for repo-wide rules
7. If docs conflict, the closer doc controls local work details, but no child doc may weaken DOX

Do not rely on memory. Re-read the applicable DOX chain in the current session before editing.

## Update After Editing

Every meaningful change requires a DOX pass before the task is done.

Update the closest owning AGENTS.md when a change affects:

- purpose, scope, ownership, or responsibilities
- durable structure, contracts, workflows, or operating rules
- required inputs, outputs, permissions, constraints, side effects, or artifacts
- user preferences about behavior, communication, process, organization, or quality

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Victozee26/acode-opencode](https://github.com/Victozee26/acode-opencode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
