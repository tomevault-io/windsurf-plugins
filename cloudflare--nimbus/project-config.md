---
trigger: always_on
description: > Read this first if you're an AI agent picking up work on the Nimbus codebase. Sister file [`CLAUDE.md`](./CLAUDE.md) mirrors this content — keep them in sync.
---

# Nimbus — agent context

> Read this first if you're an AI agent picking up work on the Nimbus codebase. Sister file [`CLAUDE.md`](./CLAUDE.md) mirrors this content — keep them in sync.

## What this project is

Nimbus builds documentation sites on Astro. The architecture splits into three tiers:

- **User-owned starter files** — visible UI components, layouts, and styling. Copied into the user's repo by `create-nimbus-docs` and edited freely from then on.
- **`nimbus-docs` npm package** — invisible plumbing (data helpers, validation, integration wiring, behavior primitives). Imported, not forked.
- **Registry** — optional components, utilities, and agent-handoff features installed on demand via `nimbus-docs add <slug>`.

Cloudflare is a first-class deploy target (the scaffolder defaults to it and ships `wrangler.jsonc`), but the framework is deploy-target agnostic — static output runs anywhere.

## Repo layout

```
monorepo/
├── packages/
│   ├── nimbus-docs/                       framework — integration, helpers, schemas, types, `nimbus` CLI
│   ├── nimbus-starter-source/             canonical source — fat tree; doubles as kitchen-sink dev app
│   │   ├── src/                           components, layouts, pages (incl. pages/dev/), demo content
│   │   ├── templates/                     per-variant content overrides (empty/, …)
│   │   └── starter.manifest.mjs           declarative generation policy (registry-only slugs, dev-only paths, variants)
│   └── create-nimbus-docs/                scaffolder (`pnpm create @cloudflare/nimbus-docs`) — CLI only, no templates
│       └── scripts/copy-template.mjs      generator: canonical source + manifest → variant dirs (--out)
├── apps/
│   └── www/                               docs site + registry hosting
│       └── registry/                      manifests.ts (source), components/, features/, registry.json
├── examples/
│   └── local/                             local sandbox (not drift-mirrored)
├── scripts/
│   ├── release.mjs                        release orchestration (detect → generate → verify → sync+tag → publish)
│   ├── sync-templates-repo.mjs            sync generator output to the orphan templates branch + tag templates-v<version> (idempotent)
│   ├── templates-check.mjs                PR CI: generate + scaffold + build
│   ├── check-no-major.mjs / freshness-guard.mjs  release guards
│   ├── local.mjs / local-add.mjs          local sandbox helpers
├── .generated/                            gitignored generator output (templates); scratch for local/CI/release
├── pnpm-workspace.yaml
└── tsconfig.base.json
```

## Build / dev / test

```sh
pnpm -r build                                    # build all packages and apps
pnpm --filter nimbus-docs build                  # framework only
pnpm --filter nimbus-docs typecheck              # tsc --noEmit
pnpm --filter nimbus-starter-source build        # build the canonical source (kitchen-sink)
pnpm --filter nimbus-starter-source dev          # run kitchen-sink dev server (every component visible)
pnpm dev                                         # alias for the above
pnpm build:templates                             # generate template variants into .generated/templates
pnpm templates:check                             # generate + scaffold + build one variant (CI runs on relevant PRs)
pnpm local                                       # spin up the local sandbox (generates + scaffolds offline)
```

Root `build` runs at default concurrency; `pnpm -r` topo order builds `nimbus-docs` first. `apps/www`'s `build` no longer builds `nimbus-docs`, so a bare `pnpm --filter @nimbus/www build` on a clean checkout fails — deploy via `pnpm run deploy` (its `predeploy` builds the framework) or root `pnpm build`.

## The boundary test (read before adding any file)

The architecture splits into three tiers, one test per tier:

| Tier | Lives in | Test |
|---|---|---|
| **Framework** | `packages/nimbus-docs/` | *"If I edit this, am I changing taste or fixing a bug?"* Bug = framework. |
| **Starter source** | `packages/nimbus-starter-source/` | *"Do edits change Tailwind classes or layout, or do they change call signatures?"* Tailwind/layout = starter source. |
| **Registry** | `apps/www/registry/` | *"Does every docs site need this on day 1?"* No = registry, install via `nimbus-docs add`. |

**When in doubt, default to framework; the starter should grow slowly.**

## Derived templates


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cloudflare/nimbus](https://github.com/cloudflare/nimbus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
