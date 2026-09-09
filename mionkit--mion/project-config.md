---
trigger: always_on
description: > ⚠️ When replying to the user, talk in plain everyday language and extremely condensed phrases! Avoid jargon and internal nicknames unless very basic. If a term or idea could be unclear, define it in one short sentence, and add a tiny code example when it helps.
---

# Mion & RunTypes Guidelines

> ⚠️ When replying to the user, talk in plain everyday language and extremely condensed phrases! Avoid jargon and internal nicknames unless very basic. If a term or idea could be unclear, define it in one short sentence, and add a tiny code example when it helps. 
> Never use em dashes "—" 
> Do not load linked / relevant files into context unless the current task strictly needs them!

For setup, build, test, and publish workflows, see [SETUP.md](SETUP.md), the single setup document.
If environment is not already setup you can run the [mion-setup skill](.claude/skills/ts-runtypes-setup/) — it drives the whole host bootstrap end-to-end. Don't hand-roll a bootstrap! 

## ⚠️ IMPORTANT!!! any issue found during a task must be FIXED, not filed for later

This is the rule broken most often, so it comes first!! Any issue or blocker you discover while doing a task should be fixed before task gets done:

- **Related to the current task** → fix it in the SAME task and the SAME pull request, with its own commit and its own test. Size buys no exemption — a big related finding means a bigger PR, not a later one.
- **Completely Unrelated to the current task** → delegate it to a PARALLEL background agent, never a backlog — run the [delegate-finding skill](.claude/skills/delegate-finding/). That takes care of creating the todo and delegating it to a parallel agent.
- A [docs/todos/](docs/todos/) spec is a commitment to solve it, never a way to close the loop!

**Absolute:** never let a finding slide and get lost, either fix it or delegate it to a parallel claude session. Ask if there are open questions you can't solve!

## Setup

- Go ≥ 1.26
- Node ≥ 26
- podman ≥ 4.0
- git
- pnpm ≥ 11 — never `npm install`; workspace policies live in `pnpm-workspace.yaml` (`.npmrc` is auth/registry only, everything else is silently ignored there).


## JS monorepo (`packages/`)

One pnpm workspace, one `@mionjs/*` namespace, one release train: the type-system packages and the framework packages ride the same `version.json` lockstep and depend on each other via `workspace:*`. 
All `dependencies` / `devDependencies` are exact-pinned. THREE peerDeps exceptions stay as ranges: `@mionjs/devtools` (so consumers can dedupe Vite) and, on the `@mionjs/drizzle-orm-*-core` packages, BOTH their `drizzle-orm` peer (the range IS the compatibility promise of their drizzle-aligned version line) and their `@mionjs/run-types` peer (the consumer's single copy must supply both the format types and the runtime `getRunType` the tableFromType/toDrizzle marker overloads forward to; an exact pin would also force a republish every release).
Cross-package deps use the `workspace:*` protocol. All devDependencies live root-level, never per-package, with ONE exception: each drizzle dialect package carries `@mionjs/run-types: workspace:*` as a devDependency to satisfy its own peer in the workspace (dev deps never reach a consumer). 

- [mion](packages/run-types/) — public marker + runtime helpers (`InjectRunTypeId<T>`, `InjectTypeFnArgs<T,Fn>`, `getRunTypeId`, runtime family bodies).
- [@mionjs/devtools](packages/devtools/) — build-tool integration around the resolver, and the ONLY devtools package (the two merged). Five source areas: `src/core/` (bundler agnostic), `src/runtypes/` (the unopinionated adapter per bundler), `src/vite/` and `src/next/` (the mion presets, one dir per host, sharing `src/options.ts`), `src/lint/`. Read [its CLAUDE.md](packages/devtools/CLAUDE.md). What it does:
  - **Transform** — rewrites `createX<T>()` call sites and injects the import block.
  - **Codegen** — emits per-entry cache modules under `<genDir>/types/`.
  - **Enrich** — scaffolds and keeps in sync the FriendlyText + MockData mirror files.
  - **Lint** — ONE module, TWO namespaces on the `./eslint` (and `./oxlint`) subpath: the default export is the `runtypes/*` plugin OXlint's `jsPlugins` loads, `mionPlugin` carries mion's own `@mionjs/*` rules, and `configs.recommended` registers both for ESLint. ESM only, deliberately: its top-level `await prewarmSession()` has no CommonJS spelling and must fork the resolver launcher before OXlint reserves its address space.
  - **Presets** — `@mionjs/devtools/vite` (`mionVitePlugin`) and `@mionjs/devtools/next` (`withMion`) are the mion-opinionated entries; the plain adapters live under `@mionjs/devtools/runtypes/*` (vite, rollup, rolldown, webpack, rspack, esbuild, bun, next). Both presets map options through `src/mion/options.ts` so they cannot drift.
  - ⚠️ **Next.js / Turbopack** ([src/runtypes/next/](packages/devtools/src/runtypes/next/)) — the one adapter reaching a bundler with NO plugin API: a broker started from `next.config` plus a `turbopack.rules` loader. `withMion` composes those pieces; it never nests one wrapper in another.
    Read [src/runtypes/next/CLAUDE.md](packages/devtools/src/runtypes/next/CLAUDE.md) first, it records invariants that look like cleanups but are not!

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MionKit/mion](https://github.com/MionKit/mion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
