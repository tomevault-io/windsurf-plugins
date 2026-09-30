---
trigger: always_on
description: This file is the repo law for agents and contributors. It holds commands, validation rules, architecture constraints, and project stop rules. Read `VISION.md` for intent before planning substantial work. `VISION.md` does not override this file. Pi sessions also load `.pi/APPEND_SYSTEM.md` and the repo-local extension `.pi/extensions/project.ts`.
---

# Agent instructions

This file is the repo law for agents and contributors. It holds commands, validation rules, architecture constraints, and project stop rules. Read `VISION.md` for intent before planning substantial work. `VISION.md` does not override this file. Pi sessions also load `.pi/APPEND_SYSTEM.md` and the repo-local extension `.pi/extensions/project.ts`.

## Stack contract

Workspace `package.json` files declare the pinned stack. [README.md](./README.md#what-is-in-the-stack) summarizes it. Keep dependencies exact. Repo-local config wins. Record drift instead of silently migrating the project.

- pnpm workspaces + Turborepo (`apps/*`, `packages/*`)
- Node `>=24.18.0` and pnpm `11.3.0`; do not replace pnpm with Bun or npm for installs
- Effect `4.0.0-rc.117` and `@effect/platform-node` `4.0.0-rc.117`
- XState `6.0.0-alpha.59` for finite lifecycles, retries, cancellation, and resumability
- `@xstate/effect` `0.1.0-alpha.2` bridges the two: machines run as scoped Effects via `createEffectActor`, side effects are declared `fromEffect` actors. Published to npm 2026-09-19; `vendor/README.md` keeps the rules for the next unpublished pin
- Alchemy `2.0.0-beta.79` (Infrastructure as Effects) for every cloud resource; declared in `apps/infra/alchemy.run.ts`, authenticated through Alchemy profiles, never through env vars in this repo
- TypeScript `7.0.2` in strict mode, patched by `@effect/tsgo` `0.45.0` in `prepare` so the Effect language service diagnostics in `tsconfig.base.json` fail `tsc`, not just the editor. Escape hatch for a real boundary: `// @effect-diagnostics-next-line <rule>:off` with a reason
- `@effect/vitest` `4.0.0-rc.117` for every Effect test: `it.effect` and `it.layer(layer)`; `Effect.run*` and `ManagedRuntime.make` in test files are a lint error
- `@oxlint/plugins` `1.83.0` for the two typed lint rules in `scripts/oxlint-plugin-*.ts` and the vendored [anti-slop](https://github.com/dmmulroy/anti-slop) rules in `tools/oxlint/anti-slop/` (all generic rules plus the Effect group, at `error`). The copy is ours; `UPSTREAM.md` there says where it came from and how to take upstream fixes
- Oxlint `1.83.0` with Ultracite `7.12.0`, Oxfmt `0.68.0`, and Turborepo `2.11.2`
- varlock `1.20.0`: declare every env var in `.env.schema`, never read `.env.local` directly, run `pnpm env:check` after schema edits

## Packages

| Package | Path | Role |
| --- | --- | --- |
| `@rat-stack/capability` | `packages/capability` | `defineContract`, `implement`, and the `toCommand`, `toHttpApi`, `toToolkit`, `toRpc`, and `toCodeMode` projections |
| `@rat-stack/core` | `packages/core` | Shared `inspectFile`, `search`, and `read` contracts; `inspectFile` handler, lifecycle machine, and `FileInspector` |
| `@rat-stack/database` | `packages/database` | Database cartridge: the `RunLog` service and the `DatabaseVendor` choice (D1 or Hyperdrive Postgres), with each vendor's schema, migrations, and resources |
| `@rat-stack/auth` | `packages/auth` | Better Auth cartridge over the chosen `DatabaseVendor`: `Auth`, `CurrentPerson`, and the RPC middleware that provides it; only `Unauthenticated` crosses the wire. `@rat-stack/auth/devtools` adds `runAsPerson`, the dev-only `rat_test_person` capability, and `testPersonLayer` |
| `@rat-stack/devtools` | `packages/devtools` | 🐀 devtools cartridge: `CallLog`, `record`, and the `rat_*` capabilities that list, read, dispatch, replay, and diff capability calls |
| `@rat-stack/web` | `apps/web` | Default UI bin: TanStack Start on Vite, deployed as `Website` (`apps/web/src/website.ts`). Browser `client/` builds AtomRpc from contracts; `/rpc` uses the `#backend` import: production forwards to the private `RpcBackend` in `apps/mischief/src/rpc-worker.ts`, and `pnpm --filter @rat-stack/web dev` serves the content capabilities in process from `src/dev/backend.ts` (the `development` condition in `apps/web/package.json`). In dev, `src/dev/backend.ts` runs the content capabilities under devtools: `/rpc` calls are recorded, `/__rat/rpc` serves the 🐀 overlay (`#devtools-overlay`, `src/dev/features/overlay`, ⌘K), and `/__rat/mcp` serves every `rat_*` tool to agents, with memory-Auth test people. `src/dev/client` and `src/dev/features` follow the browser rules. `test/production-bundle.test.ts` proves no `src/dev` module, no devtools code, and no test people reach a production build. Include it in the Stack with `const website = yield* Website;` |
| `@rat-stack/cli` | `apps/cli` | Composition root: `stats`, `catalog`, `openapi`, `serve [--devtools]`, `mcp [--code-mode] [--devtools]` commands |
| `@rat-stack/infra` | `apps/infra` | Alchemy Stack: the project's cloud footprint as one Effect program |
| `@rat-stack/mischief` | `apps/mischief` | Cloudflare Worker: the public site, agent discovery, and sandboxed execute surface |

## Nouns

| Noun | Home | Runtime job |
| --- | --- | --- |
| Contract | `packages/core/src/contracts.ts` | Shared name, schemas, failure, annotations, and approval setting. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joelhooks/rat-stack](https://github.com/joelhooks/rat-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
