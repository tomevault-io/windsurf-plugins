---
trigger: always_on
description: shadcn M3E: Material 3 Expressive (M3E) on the shadcn/ui template. React 19, TanStack Router, Vite (via `vite-plus`), Tailwind CSS v4, Base UI, bun. It is both a component library (distributed as a shadcn registry) and the documentation site for it.
---

# AGENTS.md

shadcn M3E: Material 3 Expressive (M3E) on the shadcn/ui template. React 19, TanStack Router, Vite (via `vite-plus`), Tailwind CSS v4, Base UI, bun. It is both a component library (distributed as a shadcn registry) and the documentation site for it.

- Site and registry: https://shadcn-m3e.crystaworld.dev (registry at `/r/{name}.json`, namespace `@m3e`)
- Repository: Crysta1221/shadcn-m3e

## Commands

Run everything from the repository root; the root scripts delegate to the workspaces.

```bash
bun run dev             # docs site on http://localhost:5173 (rebuilds the registry first)
bun run typecheck       # tsc -b --noEmit (apps/docs and packages/m3e)
bun run lint            # oxlint, type-aware
bun run format          # oxfmt (format:check to verify only)
bun run registry:build  # apps/docs/public/r + apps/docs/src/docs/registry-meta.generated.ts
bun run gen:icons       # bundle the icons the code uses; check:icons verifies (CI)
bun run gen:tokens      # regenerate packages/m3e/src/styles/m3e.generated.css
bun run build           # tsc -b && vp build
bun run dev:canvas      # the Playground (apps/canvas) on http://localhost:3000
```

Before you finish a change: `typecheck`, `lint`, `format`, and `registry:build` when a file in `packages/m3e/src` changed. There is no test runner; UI is verified in the browser (see "Verifying").

## Deployment (Cloudflare)

Each app deploys as its own Worker via Workers Builds (repo connected once per Worker; see the dashboard settings below — root directory, commands and watch paths live in the dashboard, not in the repo).

- `apps/docs` → Worker `shadcn-m3e-docs`. Configured for `cf` (the new CLI, beta): `apps/docs/cloudflare.config.ts` (Worker name, SPA `notFoundHandling`, observability) plus `apps/docs/wrangler.config.ts` (`assetsDirectory: "dist"` — cf has no `assets.directory` field; static sites keep that in the Wrangler build config). `cf`/`wrangler` are devDependencies of `apps/docs`. Deploys `bun run --cwd apps/docs deploy` (`cf deploy`), previews `deploy:preview` (`cf previews deploy`).
- `apps/canvas` → Worker `m3e-canvas`. A Vite+ SPA (`vp build` → `dist/`) deployed with the Wrangler CLI: `apps/canvas/wrangler.jsonc` + `bun run deploy:canvas` (root script). No entrypoint — assets only, SPA fallback.
- `cf` runs under Node.js (loading `cloudflare.config.ts` on Bun fails) and requires Node 22.18+. On Windows, `cf deploy` currently fails at `spawn EFTYPE` — a cf beta bug: it spawns `wrangler/bin/cf-wrangler.js` directly instead of through `node`. Works on Linux, so Workers Builds is unaffected; locally `node node_modules/wrangler/bin/cf-wrangler.js build` produces `.cloudflare/output/v0` the same way.
- `devEngines` pins bun, which makes `npm`/`npx` fail inside the repo — CI commands must go through `bun run` scripts, never `npx`.

Workers Builds dashboard settings:

| Worker          | Root directory | Build command                             | Deploy / preview command                                                    | Build watch paths (includes)                                                 |
| --------------- | -------------- | ----------------------------------------- | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| shadcn-m3e-docs | `/`            | `bun run registry:build && bun run build` | `bun run --cwd apps/docs deploy` / `bun run --cwd apps/docs deploy:preview` | `apps/docs/*`, `packages/m3e/*`, `package.json`, `bun.lock`, `tsconfig.json` |
| m3e-canvas      | `/`            | `bun run build:canvas`                    | `bun run deploy:canvas`                                                     | `apps/canvas/*`, `package.json`, `bun.lock`, `tsconfig.json`                 |

The root directory stays at `/` because `bun install` runs at the workspace root. Watch paths use dashboard wildcards (`*` crosses `/`, so nested paths are covered); `packages/m3e` is included for docs because the site builds against it.

## Repository layout

A bun workspace. `apps/*` are applications, `packages/*` is the code that ships.

```
apps/docs/       the documentation site; it also serves the registry at /r/{name}.json
apps/canvas/     the Playground: TanStack Router + Vite+, forked from lnkiai/m3e-canvas
packages/m3e/    the M3E components, library code and tokens (the registry source)
videos/          a separate Remotion project, not part of the workspace
```

The code in `packages/m3e` imports its own files as `@/components/m3e/x`, `@/lib/m3e/x`, `@/hooks/x` and `@/styles/x`: those are the specifiers a consumer's project has after `shadcn add`, and `scripts/build-registry.mjs` reads them. `apps/docs/tsconfig.paths.json` and `apps/docs/vite.config.ts` map them (and `@/` itself, to `apps/docs/src`) to the right directory. Keep the two in sync. The registry still publishes the consumer-side paths (`src/components/m3e/…`), see `SOURCES` in the script.

## Directories


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Crysta1221/shadcn-m3e](https://github.com/Crysta1221/shadcn-m3e) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
