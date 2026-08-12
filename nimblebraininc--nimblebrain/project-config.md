---
trigger: always_on
description: > This file is read by agents. Keep edits terse, imperative, token-aware. No long-form prose; bullets with concrete triggers and examples.
---

# NimbleBrain

> This file is read by agents. Keep edits terse, imperative, token-aware. No long-form prose; bullets with concrete triggers and examples.

Self-hosted platform for MCP Apps and agent automations, built on Bun. Agentic loop + MCP bundle management + interactive UI host + cron-scheduled automations + skill-driven prompt composition + HTTP API + web client.

## Build & Verify

```bash
bun install                # Install dependencies
bun run dev                # API (:27247) + Web (:27246) with watch/HMR
bun run dev:worktree       # Run from any worktree against an isolated workdir on alt ports — see "Worktree dev" below
bun run dev:api            # API only with auto-restart
bun run verify             # Full CI parity — runs every subscript below
bun run verify:static      # format:check + lint + check + check:cycles
bun run verify:test-unit   # test:unit + test:web + test:bundles

bun run test               # Unit then integration (stops at the first failing suite)
bun run test:unit          # Unit tests only (fast, ~10s)
bun run test:integration   # Integration tests only
bun run lint               # Biome linter
bun run format:check       # Biome format diff (no writes) — matches CI
bun run check              # TypeScript strict mode
bun run format             # Biome auto-format (writes)

cd web && bun install      # Web client dependencies (separate package.json)
cd web && bun run build    # Web production build → web/dist/

bun run install:bundles    # Bundle UI deps (each a separate package.json) — the exact command CI runs
bun run build:bundles      # Rebuild every src/bundles/*/ui (vite single-file)
```

**A fresh checkout/worktree must install `web/` AND every `src/bundles/*/ui/` before `bun run verify`.** `verify:test-unit` runs `test:web` + `test:bundles`, which execute those separate packages; root `bun install` doesn't cover them, so verify fails with a missing-module error (e.g. `Cannot find package 'dompurify'`) until they're installed. **But `test:unit` itself runs on root deps alone** — the backend unit suite imports the shared bridge protocol (`web/src/bridge/*`), so a web-only *value* import must never leak into that graph: keep such deps type-only and inject the value at the browser entry (`web/src/sentry.ts` is the pattern). The `Unit Tests (root deps only)` CI job enforces this; only `test:web`/`test:bundles` need the `web/` + bundle installs.

**A fresh checkout prepares itself.** `node_modules` and `dist/` are both gitignored, so a
new clone or worktree has neither. Every dev launcher — `dev`, `dev:empty`, `dev:minimal`,
`dev:docs-demo`, `dev:worktree` — installs `web/` dependencies and builds any bundle UI
missing its `dist/index.html` before starting, so the quickstart does not need those steps.
`dev:worktree` additionally installs **root** dependencies, which it must: `scripts/dev.ts`
imports from `src/`, so it cannot install the dependencies it needs in order to load. Only
what is absent is done — see the rebuild note below.

**`bun run dev` does NOT rebuild bundles.** The API serves each bundle from its pre-built `src/bundles/<name>/ui/dist/index.html`. After editing any file under `src/bundles/*/ui/src/`, run `bun run build:bundles` and restart the dev server (the API reads dist on iframe mount; it doesn't watch the file). Forgetting this means the iframe loads stale code while your changes look "live" in the source tree — a high-confusion failure mode.

**Before opening a PR, run `bun run verify`.** It is the single command that mirrors CI, enforced by construction: `.github/workflows/ci.yml` invokes only `verify:*` subscripts (plus `test:integration`) — no inline check steps. To add or change a check, edit the matching subscript in `package.json`; CI picks it up automatically. If CI ever catches something `verify` didn't, the fix is to update the subscript, not the checklist. Tool-level parity is the gate; discipline-level rules are not.

### Worktree dev

`bun run dev:worktree` runs the platform from any git worktree against a worktree-local workdir, on alt ports, with no auth gate — for QA on a feature branch without disturbing your primary `~/.nimblebrain` dev or another worktree's state.

| Setting | Value |
|---|---|
| Workdir | `<worktree>/.nimblebrain-worktree/` (auto-seeded; gitignored) |
| Config | `<worktree>/.nimblebrain-worktree/nimblebrain.json` (auto-seeded on first run) |
| API / Web ports | 27271 / 27270 (override via `NB_API_PORT` / `NB_WEB_PORT`) |
| Auth | none (dev mode — no `instance.json`) |
| LLM keys | `ANTHROPIC_API_KEY` (and friends) read from your shell environment |

Each worktree gets its own isolated state, so two worktrees can run side-by-side without colliding. Reset with `rm -rf .nimblebrain-worktree && bun run dev:worktree`. Share state across worktrees with `NB_WORK_DIR=/abs/path bun run dev:worktree`. Suitable for Chrome DevTools-driven E2E tests against `/v1/*` (no login dance).

## Conventions

- **Runtime:** Bun (not Node). Use `bun run`, `bun test`, `bunx`.
- **Module system:** ESM only. All imports use `.ts` extensions.
- **Linting:** Biome (not ESLint/Prettier). Run `bun run lint`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NimbleBrainInc/nimblebrain](https://github.com/NimbleBrainInc/nimblebrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
