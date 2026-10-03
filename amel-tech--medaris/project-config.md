---
trigger: always_on
description: Instructions for AI agents working in this repository. Read [`README.md`](README.md) first for the layout and commands; this file covers what is easy to get wrong.
---

# CLAUDE.md

Instructions for AI agents working in this repository. Read [`README.md`](README.md) first for the layout and commands; this file covers what is easy to get wrong.

## What this repo is

A single pnpm workspace orchestrated by Nx: two NestJS APIs (`tedrisat`, `teskilat`), five web apps, ten `@medaris/*` libraries. It was formed by merging `madrasah-backend` and `madrasah-frontend` with history preserved. Any instruction that describes two separate repositories is stale.

## Before you touch anything

- **`pnpm install` in every fresh clone and every new git worktree.** The husky dispatcher lives in git-ignored `.husky/_`; without it `commitlint` and `lint-staged` silently do not run and a bad commit lands clean.
- **`libs/common` must be built before the Nest apps start**, otherwise they fail at boot with `TS2307`.
- Use Nx **project names**, not directory names: `apps/tedris` is `tedris-web`, `apps/nizam` is `nizam-web`, `apps/nazir` is `nazir-web`, `apps/landing` is `landing-web`.

## The gate

```bash
pnpm nx run-many -t typecheck --skip-nx-cache
pnpm nx run-many -t test --skip-nx-cache
pnpm nx run-many -t build --skip-nx-cache
pnpm nx run-many -t lint --skip-nx-cache
pnpm nx run-many -t module-boundaries --skip-nx-cache
```

Expected: every project green on typecheck, lint and module-boundaries; every build target green; every suite green. (Counts are deliberately not written here — they change with every project or spec added and go stale immediately; read them off the command output.)

Two prerequisites that look optional and are not:

- **`-t test` needs a running Docker daemon, plus `curl` and `jq` on the host.** `apps/tedrisat/vitest.config.ts` matches `test/**/*.spec.ts`, which includes the twenty-three `test/e2e/*.e2e.spec.ts` suites, and those start a Testcontainers `postgres:17-alpine` — `keycloak-audience.e2e.spec.ts` also starts `quay.io/keycloak/keycloak:26.3.2` (MDRS-42). Of tedrisat's 45 suites, 23 are e2e. `test:e2e` re-runs the same twenty-three under a separate config — it is not extra coverage. `keycloak-audience.e2e.spec.ts` runs `tools/keycloak/setup-realm.sh`, which exits 1 when `curl` or `jq` is missing — a missing tool, not a regression.
- **`-t build` needs the root `.env` for the four Next.js apps.** They validate the environment at build time, so a fresh worktree fails with `Invalid environment variables` until `cp .env.example .env` has been run. That is a missing file, not a regression.

- **The tedrisat suite may not reach the network (MDRS-89).** `apps/tedrisat/test/setup-no-network.ts` replaces **`globalThis.fetch`** in every worker and refuses any host that is not loopback — and records the refusal, so a caller that swallows the error still fails the test in `afterEach`. That second half is load-bearing: `KeycloakPublicKeyProvider.onModuleInit` catches its own failure and logs, which is how every `createTestApp` called the live realm for months without anything going red. Containers on loopback are fine; that is how `keycloak-audience.e2e.spec.ts` reaches its own Keycloak. **It covers `fetch` only** — `node:http`/`node:https`, `undici.request` and child processes (`tools/keycloak/setup-realm.sh` shells out to `curl`) are not seen. Widening it means an undici `setGlobalDispatcher`, which has not been done.

**`-t test` is the only gate that catches a broken NestJS container.** `typecheck` and `build` stay green while dependency injection is already broken at runtime — this has happened, see below. Never skip it.

## Traps that have already cost time

- **`useImportType` breaks NestJS DI.** Rewriting a constructor parameter type to `import type` erases it from `design:paramtypes`, and Nest can no longer resolve the dependency. 78 of 89 tests went red this way while typecheck and build were green. `biome.json` disables the rule for the three packages that use `emitDecoratorMetadata` (`apps/tedrisat`, `apps/teskilat`, `libs/common`). Do not re-enable it there.
- **`biome.json` must stay comment-free.** A comment anywhere in that file — including *above* the `overrides` array — makes Biome silently drop the override's `includes`, which re-enables `useImportType` on the Nest packages. Measured, not theorised. Put explanatory notes in `docs/migration/` instead.
- **Do not partially stage a file the formatter will reflow.** `git add -p` plus `biome check --write` re-applies the hidden hunks at a stale offset and can produce a syntax error while lint-staged still exits 0. Stage whole files.
- **`NODE_ENV=development` in a web app's `.env` breaks `next build`.** React resolves its development bundle against a production SSR runtime and the build dies prerendering `/_global-error` with `Cannot read properties of null (reading 'useContext')`. Measured on all four web apps. The root `.env.example` therefore scopes `NODE_ENV` to `API__`; never broadcast it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amel-tech/medaris](https://github.com/amel-tech/medaris) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
