---
trigger: always_on
description: > **Note:** Claude Code automatically loads this file.
---

# Arkor Development Guide

> **Note:** Claude Code automatically loads this file.

## Repository shape

pnpm + Turbo monorepo. Workspaces are declared in `pnpm-workspace.yaml` (`packages/*`, `e2e/*`, `examples/*`).

| Path | Role |
| --- | --- |
| [packages/arkor](packages/arkor) | Published `arkor` SDK + CLI + bundled local Studio server (Hono). `bin/arkor` → `dist/bin.mjs`. Library entry → `dist/index.mjs`. |
| [packages/create-arkor](packages/create-arkor) | Published `create-arkor` scaffolder (`pnpm create arkor`). |
| [packages/cli-internal](packages/cli-internal) | **Private** workspace package. Source is bundled into `arkor` and `create-arkor` via tsdown's `deps.alwaysBundle`. Never appears as a runtime dependency on npm. |
| [packages/studio-app](packages/studio-app) | **Private** Vite + React 19 SPA. `pnpm --filter @arkor/studio-app bundle` builds it; `packages/arkor/scripts/copy-studio-assets.mjs` copies `dist/` into `packages/arkor/dist/assets/`. |
| [e2e/cli](e2e/cli) | **Private** vitest suite that spawns the built `dist/bin.mjs` of both CLIs in temp dirs. |
| [e2e/studio](e2e/studio) | **Private** Playwright suite that spawns `arkor dev` against an in-process fake cloud-api and drives the Studio SPA in Chromium. |
| [examples/doc-drift](examples/doc-drift) | **Private** use-case example: documentation drift check against an Arkor deployment (zero-dependency script + copy-me workflow). |
| [docs](docs) | Source for [docs.arkor.ai](https://docs.arkor.ai). |

## Common commands

Root scripts fan out via Turbo (which respects `^build` deps in [turbo.json](turbo.json)):

```bash
pnpm install
pnpm build          # turbo run build across all packages
pnpm typecheck      # tsc --noEmit across all packages
pnpm lint           # oxlint --deny-warnings, then strict ESLint 10 (single root config each)
pnpm format         # oxfmt --write across the repo (config in oxfmt.config.ts)
pnpm format:check   # oxfmt --check; CI gates on this (no writes)
pnpm test           # vitest run across all packages (incl. e2e)
pnpm test:coverage  # writes lcov + cobertura + junit per package; CI uploads lcov to Codecov, cobertura to GitHub native code coverage, junit to Codecov Test Analytics
```

Per-package iteration:

```bash
pnpm --filter arkor dev                # tsdown --watch on the SDK/CLI
pnpm --filter @arkor/studio-app dev    # Vite dev server (5173, proxies /api → :4000)
pnpm --filter create-arkor dev         # tsdown --watch on the scaffolder
pnpm --filter @arkor/e2e-cli test      # E2E (slow; spawns real CLIs)
SKIP_E2E_INSTALL=1 pnpm --filter @arkor/e2e-cli test   # skip `<pm> install` inside fixtures
ARKOR_E2E_PM=bun pnpm --filter @arkor/e2e-cli test     # run the install-matrix case for one pm only (CI sets this per-runner; valid labels: npm / pnpm / yarn / yarn-berry / bun)
pnpm --filter @arkor/e2e-studio exec playwright install chromium   # one-time browser install (run first on a fresh checkout)
pnpm --filter @arkor/e2e-studio test   # Studio E2E in Chromium (Playwright)
```

Run a single test file: `pnpm --filter <pkg> exec vitest run path/to/file.test.ts`. Use `vitest run -t "name"` to filter by test name.

Trying a local build end-to-end (the loop CONTRIBUTING.md recommends):

```bash
pnpm build
node packages/create-arkor/dist/bin.mjs my-arkor-app   # in a scratch dir
cd my-arkor-app && pnpm dev                            # Studio at http://127.0.0.1:4000
```

## Architecture notes that span files

### CLI build outputs

[packages/arkor/tsdown.config.ts](packages/arkor/tsdown.config.ts) emits two entries (`bin.mjs`, `index.mjs`). Nothing in `src/index.ts` imports the Studio server, so the **entire Studio server is bundled into `dist/bin.mjs`** rather than into a separate file. When tracing Studio behaviour in a built tarball, look for the bin, not `index.mjs`.

`tsdown` also defines `__SDK_VERSION__`, `__ARKOR_POSTHOG_KEY__`, and `__ARKOR_POSTHOG_HOST__` at build time. The fallback in `core/version.ts` only fires under vitest where the transform doesn't run.

### Studio CSRF token (security-critical)

`arkor dev` generates a 32-byte base64url token per launch ([packages/arkor/src/cli/commands/dev.ts](packages/arkor/src/cli/commands/dev.ts)) and:

1. Passes it to `buildStudioApp({ studioToken })`. The Hono server validates every `/api/*` request via `X-Arkor-Studio-Token` header (or `?studioToken=` query for `EventSource`, which can't set headers). Comparison uses `timingSafeEqual`.
2. Persists it to `~/.arkor/studio-token` (mode 0600) so the SPA dev workflow (`pnpm --filter @arkor/studio-app dev`) can read it via the `arkor-studio-token` Vite plugin in [packages/studio-app/vite.config.ts](packages/studio-app/vite.config.ts), which injects `<meta name="arkor-studio-token">` into `index.html` on each request. Persistence failure must NOT block server start (read-only `$HOME` on Docker, etc.); just warn.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arkorlab/arkor](https://github.com/arkorlab/arkor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
