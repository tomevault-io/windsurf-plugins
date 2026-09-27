---
trigger: always_on
description: Open-source monorepo for the `@eleven-labs/nest-profiler` ecosystem: a Symfony Web Profiler-inspired toolkit for NestJS. It ships 15 publishable packages (`@eleven-labs/nest-profiler` core + 14 collector packages), a consuming example app (`example-api`), shared `@repo/*` workspace presets, an English-only Fumadocs site, and full CI / release automation to publish to npm.
---

# nest-profiler

## Project overview

Open-source monorepo for the `@eleven-labs/nest-profiler` ecosystem: a Symfony Web Profiler-inspired toolkit for NestJS. It ships 15 publishable packages (`@eleven-labs/nest-profiler` core + 14 collector packages), a consuming example app (`example-api`), shared `@repo/*` workspace presets, an English-only Fumadocs site, and full CI / release automation to publish to npm.

Constraints:

- Open source, MIT licensed.
- Only Node `>=22.0.0` is supported. No legacy Node 20, no `.nvmrc`.
- Public package APIs must remain importable from the package root only.
- The documentation site targets package consumers, never maintainers of the monorepo itself.

---

## Technical stack

NestJS + TypeScript packages, Jest for tests, ESLint + Prettier, Turborepo for task orchestration, Changesets for versioning and publishing, Fumadocs (Next.js + MDX) for the English-only documentation site. Exact versions live in the respective `package.json` files.

Shared ESLint / Prettier / TypeScript / Jest rules live in `packages/configs/*` and are consumed as `@repo/*` packages with `workspace:*`. Never duplicate compiler, lint, or test options downstream — extend the preset.

---

## Architecture

The agent should introspect the workspace before editing; only the non-obvious rules are listed here.

- `packages/<name>` is the only path for publishable packages. New packages mirror the shape of `packages/nest-profiler`.
- `packages/configs/*` are private `@repo/*` presets, never published.
- `examples/api` is the consumer-side demonstration. It is in `.changeset/config.json#ignore` and never enters the release flow.
- `@eleven-labs/nest-profiler-ai` is pinned to the `alpha` dist-tag through its `publishConfig.tag`. It is versioned and published by the normal Changesets flow, on its own prerelease line — see `MAINTAINERS.md`.
- `docs/` is a Fumadocs site deployed to Vercel via Vercel's Git integration (no workflow in this repo), independently of package release.
- `scripts/` holds release helpers (`changesets/*`, `absolutize-readme-images.ts`), not runtime code. Repository labels are declarative (`.github/labels.yml`) and synced by the `repo-config.yml` workflow, while milestones are managed by hand in the GitHub UI; one-time GitHub setup is documented in `MAINTAINERS.md`.

---

## Code conventions

- Public exports live exclusively in each package's `src/index.ts`. Deep imports from `dist/` or internal paths are not part of the public API.
- Every public symbol that should appear in the API reference carries TSDoc comments — `<AutoTypeTable>` reads them.
- NestJS modules use `ConfigurableModuleBuilder` with `forRoot` / `forRootAsync`. Options expose an `isGlobal` flag forwarded as `global` on the returned `DynamicModule`.
- Publishable packages declare `"type": "commonjs"`, `"sideEffects": false`, `"engines.node": ">=22.0.0"`, and treat NestJS + `reflect-metadata` + `rxjs` as **peer** dependencies (and as devDependencies for local tests).
- Tests live next to the source as `*.spec.ts` and run under Jest with `ts-jest`.
- Documentation content is written in English only. Consumer documentation files use the explicit `.en.mdx` suffix, and the default language served by the site is `en`.
- Any user-visible change to a publishable package requires a Changeset entry (`pnpm changeset`). `example-api` and `@repo/*` are excluded from publishing.

---

## Available commands

The agent should read `package.json` for the full list of scripts. The conventions below describe when to use them, not which exist.

- Targeted iteration: `pnpm --filter @eleven-labs/<name> <script>` (e.g. `pnpm --filter @eleven-labs/nest-profiler test`).
- Cross-cutting actions live in root `package.json` scripts (docs, example app, release, packing). Prefer them over recreating their command lines.
- Do not invoke `turbo run …` directly from new code or workflows — route through a root script.

### Mandatory validation before delivery

Every change must pass:

1. `pnpm format:check`
2. `pnpm lint`
3. `pnpm typecheck`
4. `pnpm test`
5. `pnpm build`

Add when the change touches the matching area:

- `pnpm docs:build` — when editing anything under `docs/`.
- `pnpm pack:dry-run`, `pnpm publint`, and `pnpm attw` — when editing a publishable package's manifest or its public exports.
- Repeat the relevant suite under `nvm use 22` then `nvm use 24` when the change is runtime-sensitive.

No change is considered ready while any required step fails.

---

## Best practices

- **SOLID and clear boundaries**: each NestJS module exposes one responsibility, services are injected via tokens, options flow through `ConfigurableModuleBuilder`. Avoid generic helper layers until two packages prove they are needed.
- **DRY across the workspace**: shared lint / format / TypeScript rules live in the `@repo/*` presets, never duplicated downstream. Shared release filters live in root `package.json` scripts, not inside workflows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eleven-labs/nest-profiler](https://github.com/eleven-labs/nest-profiler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
