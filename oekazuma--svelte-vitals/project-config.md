---
trigger: always_on
description: Repository conventions for AI agent sessions. Read this before exploring the codebase — it exists so you don't have to rediscover (or guess) these facts every session.
---

# AGENTS.md

Repository conventions for AI agent sessions. Read this before exploring the codebase — it exists so you don't have to rediscover (or guess) these facts every session.

## What this is

svelte-vitals is a static code-health checker for SvelteKit — not a runtime Web Vitals reporter. It statically analyzes source code (resolved `<head>` metadata and component bodies) across five categories: SEO, Performance, Correctness, Security, Architecture. The project is pre-1.0 (all packages are on `0.x` versions).

## Verify commands

| Purpose        | Command              | Notes                                                                                                                                                                                                   |
| -------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Build          | `pnpm build`         | `pnpm -r build`                                                                                                                                                                                         |
| Typecheck      | `pnpm typecheck`     | `pnpm -r typecheck`                                                                                                                                                                                     |
| Test           | `pnpm test`          | `pnpm build && pnpm -r test` (vitest) — builds first because packages/cli's tests import @svelte-vitals/core from its built dist                                                                        |
| Floor smoke    | `pnpm smoke`         | needs `pnpm build` first — it runs the built `dist` under a bare `node`; locally that is the devEngines Node, not the floor, so the floor claim is what CI's `floor-smoke` job (pinned to 22.13.0) adds |
| Lint           | `pnpm lint`          | `oxlint .` + `oxfmt --check .`                                                                                                                                                                          |
| Format         | `pnpm format`        | `oxfmt --write .`                                                                                                                                                                                       |
| Publish checks | `pnpm check:publish` | publint + attw (`--profile esm-only`)                                                                                                                                                                   |

CI (`.github/workflows/ci.yml`) runs five jobs: `lint`, `check` (build + typecheck + check:publish), `test`, `floor-smoke`, `docs`. Run the relevant verify commands yourself and confirm they pass **before** claiming a task is complete.

## Package map

- `packages/core` — runtime-agnostic rule engine, scorer, and reporter (types + logic only).
- `packages/cli` — the `svelte-vitals` CLI.
- `packages/vite` — Vite/SvelteKit plugin + live dashboard; analyzes prerendered HTML during `vite build`.
- `docs` — Blume docs site (`docs/blume.config.ts`), English + Japanese (`docs/src/content/docs/` and `docs/src/content/docs/ja/`).
- `packages/cli/docs` — the handful of topics `svelte-vitals docs show <name>` prints. Edit the
  markdown, then `pnpm --filter svelte-vitals run gen:docs && pnpm format`; `packages/cli/test/docs-embed.test.mjs`
  fails the build if the committed `src/docs/generated.ts` drifts. Keep them terse and terminal-first —
  the site is the complete reference, this set is what a reader needs mid-run.
- The docs site's CLI flag-reference tables (`guides/(setup)/cli.md` and `install.md`, en+ja, between
  `<!-- cli-reference:start/end -->` markers) are generated from the gunshi arg declarations: after
  changing any flag or its description, run `pnpm --filter svelte-vitals run gen:cli-reference && pnpm format`;
  `packages/cli/test/cli-reference.test.mjs` fails the build on drift. Never edit inside the markers by hand.

The first-party GitHub Action is **not** part of this monorepo — it lives in its own repository,
[oekazuma/svelte-vitals-action](https://github.com/oekazuma/svelte-vitals-action), depending on
the published `svelte-vitals`/`@svelte-vitals/core` npm packages like any other consumer (regular
semver ranges, not workspace links). See `docs/superpowers/specs/2026-07-22-action-dist-post-merge-only.md`
for why it was split out. `packages/cli/scripts/gen-action-pin.mjs` (run manually via
`pnpm --filter svelte-vitals run update-action-pin`, not on every build) fetches that repo's
latest release into the committed `packages/cli/src/ci/action-pin.generated.ts`, which `ci
install`/`ci upgrade` bundle into scaffolded workflows.

## Hard rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oekazuma/svelte-vitals](https://github.com/oekazuma/svelte-vitals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
