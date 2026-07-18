---
trigger: always_on
description: > Simple, powerful, view-agnostic, modular and extensible router
---

# Real-Router

> Simple, powerful, view-agnostic, modular and extensible router

pnpm monorepo with 28 packages + `benchmarks/` + bare `shared/` sources (symlinked into consumers' `src/dom-utils`, `src/browser-env`, and `src/shared-ssr`, except `packages/angular` which uses a git-tracked copy of `dom-utils`) + 87 top-level example applications across `examples/web/<framework>/*`, `examples/desktop/{electron,tauri}/*`, and `examples/console/*` (+52 subgroup sub-examples in `web/<framework>/{animation,ssr,hash}-examples/*` → 139 runnable; 145 example workspace packages incl. 6 framework aggregators — see [Desktop Integration](https://github.com/greydragon888/real-router/wiki/Desktop-Integration)). Run `pnpm install` after cloning.

`shared/` is a minimal workspace entry (name, type, devDeps) with no `src/` of its own — it owns sibling directories `shared/browser-env/`, `shared/dom-utils/`, and `shared/ssr/` that are git-tracked symlink targets. This entry is required for `type-guards` resolution during bundling via symlinks. See IMPLEMENTATION_NOTES.md section "Shared Sources via Symlinks" for details.

### Shared Sources Tree

```
shared/
├── browser-env/   # History API + URL primitives — for browser/hash/navigation plugins
├── dom-utils/     # DOM helpers (links, scroll, a11y, view-transitions) — for framework adapters
└── ssr/           # SSR per-route loader plugin scaffolding — for ssr-data / rsc-server plugins
```

(Per-file contents change often — `ls shared/<dir>` for the current layout; see each consumer package's CLAUDE.md for what it pulls in.)

### Symlink Consumers

| Shared path           | Symlink alias in consumer | Consumer packages                                    |
| --------------------- | ------------------------- | ---------------------------------------------------- |
| `shared/browser-env/` | `src/browser-env`         | `browser-plugin`, `hash-plugin`, `navigation-plugin` |
| `shared/dom-utils/`   | `src/dom-utils`           | `preact`, `react`, `solid`, `svelte`, `vue`          |
| `shared/ssr/`         | `src/shared-ssr`          | `ssr-data-plugin`, `rsc-server-plugin`               |

**Any edit to `shared/browser-env/utils.ts`, `shared/dom-utils/link-utils.ts`, or `shared/ssr/createSsrLoaderPlugin.ts` propagates instantly to every consumer via its symlink** — verify with `pnpm build` across all affected packages. For `shared/ssr/` specifically, both `ssr-data-plugin` and `rsc-server-plugin` consume the same generic factory `createSsrLoaderPlugin<T>` with different type parameters (`unknown` vs `ReactNode`) and namespaces (`"data"` vs `"rsc"`) — one source of truth, two plugins; an edit that breaks one breaks the other.

`packages/angular/src/dom-utils` is **not** a symlink — it is a git-tracked copy, re-materialized from `shared/dom-utils/` by the `prebundle` npm script before every build (ng-packagr does not follow symlinks the same way tsdown does). **When editing `shared/dom-utils/*.ts`, also update `packages/angular/src/dom-utils/*.ts`** — or run `pnpm -F @real-router/angular bundle` to sync the copy. Verify with `readlink packages/angular/src/dom-utils`; returns empty.

## Toolchain Versions

`major.minor` of the key tooling/runtime, kept in context so suggested APIs, flags and config formats match what's actually installed (this stack is bleeding-edge — TS 6, ESLint 10 flat-config, Vitest 4, Turbo 2 — so defaults from training tend to lag). **When a version changes, or you notice a mismatch with the "source of truth" column, update this table.** `major.minor` only — patch drift is noise.

| Tool / runtime  | Version | Source of truth (actualize from here)                                      |
| --------------- | ------- | -------------------------------------------------------------------------- |
| Node.js         | 24.16   | CI pins major `24` (`.github/workflows/*`); no `engines`/`.nvmrc`, minor floats |
| npm             | 11.6    | bundled with Node 24. **Not used for publishing** — pnpm 11 publishes natively (OIDC + provenance). npm's only deliberate use is the consumer smoke-test (`scripts/smoke-test-packages.sh`: `npm install` to simulate a real consumer). Installs/builds/publish are pnpm |
| pnpm            | 11.9    | `packageManager` field, root `package.json`; behavioral config in `pnpm-workspace.yaml` (overrides/allowBuilds/settings — pnpm 11 no longer reads `.npmrc`/`package.json#pnpm`) |
| TypeScript      | 6.0     | root `devDependencies` (pinned exact, `save-exact`)                         |
| Vitest          | 4.1     | root `devDependencies`                                                      |
| tsdown          | 0.22    | root `devDependencies`                                                      |
| Turbo           | 2.10    | root `devDependencies`                                                      |
| ESLint          | 10.5    | root `devDependencies` (flat config)                                        |
| @changesets/cli | 2.31    | root `devDependencies`                                                      |
| Prettier        | 3.9     | root `devDependencies`                                                      |
| husky           | 9.1     | root `devDependencies` (v9 config format — `.husky/*` are plain scripts)    |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greydragon888/real-router](https://github.com/greydragon888/real-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
