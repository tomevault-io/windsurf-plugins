---
trigger: always_on
description: Monitor and control your coding agents from your phone. Touch controls for tmux over the web. Published on npm as `remobi`.
---

# remobi

Monitor and control your coding agents from your phone. Touch controls for tmux over the web. Published on npm as `remobi`.

## Architecture

Pure TypeScript + DOM API — no framework. Transpiles to JS via tsdown for npm distribution. Bundles a browser client via esbuild and serves it from Node.

## Stack

- **Node 22+** — runtime
- **pnpm** — package manager
- **esbuild** — browser client bundle
- **tsdown** — transpile TS → JS for npm publish
- **vitest** — test runner
- **TypeScript (strict)** — no `any`, discriminated unions for actions
- **Biome** — lint + format
- **happy-dom** — DOM testing
- **Hono** — HTTP + WebSocket server (`remobi serve`)
- **node-pty** — PTY bridge for `remobi serve`
- **xterm.js** — browser terminal rendering

## Key Commands

```bash
git config core.hooksPath .hk-hooks  # Run once after clone
pnpm test              # Run all tests
pnpm run test:pw       # Playwright e2e tests (chromium + webkit)
pnpm run check         # Biome lint + format check
pnpm run check:fix     # Auto-fix lint + format
pnpm run build         # Deprecated legacy command
pnpm run build:dist    # Transpile for publishing (tsdown)
```

## Local Development

From source (bundles overlay on the fly, no build step):

```bash
tsx cli.ts serve                                # localhost:7681, default tmux session
tsx cli.ts serve --port 8080 -- bash --norc     # custom port, bash instead of tmux
```

From a local build:

```bash
pnpm run build:dist && node dist/cli.mjs serve
```

## Conventional Commits

Commits must follow [Conventional Commits](https://www.conventionalcommits.org/) format, enforced by hk commit-msg hook.

- Format: `type(scope): description`
- Types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `ci`, `perf`, `style`, `build`, `revert`
- Breaking changes: include a `BREAKING CHANGE:` footer. `!` after type/scope is optional shorthand only and must be paired with the footer because semantic-release major detection relies on the footer.

**Choosing the right type matters** — it controls whether semantic-release publishes to npm:

| Type | Release | When to use |
|------|---------|-------------|
| `fix` | patch | Bug fix **visible to package consumers** (runtime behaviour, CLI output, published types) |
| `feat` | minor | New feature visible to consumers |
| `BREAKING CHANGE:` footer | major | Breaking change to public API; `!` is optional shorthand but not sufficient on its own in this repo |
| `ci` | none | CI/CD workflow changes (GitHub Actions, release config) |
| `chore` | none | Tooling, deps, repo hygiene — anything not shipped to consumers |
| `docs` | none | Documentation only |
| `refactor` | none | Code restructuring with no behaviour change |
| `test` | none | Adding or updating tests |

**NEVER use `fix` for non-consumer-facing changes.** `fix` triggers an npm release — it means a bug fix visible to package consumers (runtime behaviour, CLI output, published types). If the change only affects CI, dev tooling, tests, or repo internals, use `ci`, `chore`, or `test` instead — even if it "fixes" something. When in doubt, ask: "would a consumer notice if this change didn't exist?" If no, it's not `fix`.

## Module Layout

- `src/index.ts` — entry: waitForTerm then init overlay
- `src/config.ts` — defaults, defineConfig, deepMerge
- `src/types.ts` — all shared types
- `src/toolbar/` — toolbar DOM + button definitions
- `src/drawer/drawer.ts` — command drawer with flat grid
- `src/drawer/commands.ts` — re-exports defaultDrawerButtons from config
- `src/gestures/` — swipe, pinch, scroll detection + gesture lock
- `src/controls/` — font size, help overlay, combo picker, floating buttons, scroll buttons
- `src/theme/` — catppuccin-mocha + apply
- `src/viewport/` — height management, landscape detection
- `src/util/dom.ts` — element creation helpers
- `src/util/terminal.ts` — sendData, resizeTerm, waitForTerm
- `src/util/haptic.ts` — vibration feedback
- `src/util/keyboard.ts` — isKeyboardOpen, conditionalFocus
- `src/util/tap.ts` — onTap: touch + click handler for iOS Safari compatibility
- `src/util/node-compat.ts` — sleep, readStdin, spawnProcess, collectStream
- `src/actions/registry.ts` — action dispatch + clipboard
- `src/hooks/registry.ts` — lifecycle hook system
- `src/config-schema.ts` — Valibot validation schemas
- `src/config-resolve.ts` — button array resolution
- `src/config-validate.ts` — config assertions
- `src/cli/args.ts` — CLI argument parsing
- `src/pwa/` — PWA manifest, meta-tags, icons
- `src/reconnect.ts` — connection loss overlay
- `src/overlay-entry.ts` — IIFE entry point for browser bundle
- `styles/base.css` — all CSS
- `cli.ts` — CLI: serve, init, deprecated build/inject, --version
- `build.ts` — browser client bundling + HTML rendering

## Publishing

- Transpiles to JS via tsdown: `bin` → `dist/cli.mjs`, `exports` → `dist/*.mjs` + `dist/*.d.mts`
- `files` array controls what's published: `dist/`, `styles/`, `src/pwa/icons/`, `README.md`, `CHANGELOG.md`, `LICENSE`
- CI: `.github/workflows/ci.yml` — pnpm test + biome check
- Release: `release` job in `.github/workflows/ci.yml` — semantic-release on push to `main` and `dev`, gated on `check` job

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [connorads/remobi](https://github.com/connorads/remobi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
