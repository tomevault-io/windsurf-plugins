---
trigger: always_on
description: Dual-mode GitHub Action for Cloudflare Pages. **deploy** creates deployments via the Wrangler CLI and links them to GitHub Deployments/Environments; **delete** batch-removes old deployments. TypeScript ESM, GraphQL-typed GitHub API, vitest.
---

# GitHub Actions Cloudflare Pages — AI Agent Instructions

Dual-mode GitHub Action for Cloudflare Pages. **deploy** creates deployments via the Wrangler CLI and links them to GitHub Deployments/Environments; **delete** batch-removes old deployments. TypeScript ESM, GraphQL-typed GitHub API, vitest.

## Critical Rules

Non-negotiable. Violating these breaks the build or the type system.

1. **GitHub API → GraphQL only, never REST.** Read [bin/download/](bin/download/) for the request pattern before writing any GitHub call.
2. **After editing any GraphQL operation → run `pnpm run codegen`** before type-checking or building — generated types do not exist until you do.
3. **Imports → `@/` path aliases.** Keep [tsconfig.json](tsconfig.json) `paths` in sync with [vitest.config.ts](vitest.config.ts) `resolve.alias`, or `vi.mock()` silently fails.
4. **Never hand-edit [`__generated__/`](__generated__/)** — it is rebuilt by codegen.
5. **No `console.log`** — use `@actions/core` methods (`info`, `debug`, `warning`, `error`, `setFailed`).
6. **Touch an exported function → update its tests.** Tests for `bin/` scripts live in `__tests__/scripts/` (NOT `__tests__/bin/`, which vitest excludes).
7. **Change a GraphQL selection set → update every test mock** for that operation (`grep` the operation name across `__tests__/`; multiple files may mock it).
8. **Run scripts with the right runner**: `node` runs scripts whose only **runtime** imports are `node:*` / relative / `package.json` / npm packages / type-only `@/` aliases (type-only imports are erased, so they're fine). `tsx` is required for any script that **transitively imports the generated GraphQL client** (`__generated__/gql/`) — see [Build & Tooling](#build--tooling).

## Architecture

| Path                                             | Role                                                                                                                        |
| ------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| [src/deploy/index.ts](src/deploy/index.ts)       | Deploy entry → bundled to [dist/deploy/index.js](dist/deploy/index.js) → consumed by [action.yml](action.yml)               |
| [src/delete/index.ts](src/delete/index.ts)       | Delete entry → bundled to [dist/delete/index.js](dist/delete/index.js) → consumed by [delete/action.yml](delete/action.yml) |
| [src/common/](src/common/)                       | Shared logic for both actions (GitHub API, Cloudflare deploy, batch ops)                                                    |
| [src/common/github/](src/common/github/)         | GitHub API: deployments, comments, environments                                                                             |
| [src/common/cloudflare/](src/common/cloudflare/) | Cloudflare Pages API + deployment logic                                                                                     |
| [`__generated__/`](__generated__/)               | Generated — never edit (gql types, webhook payloads, API responses)                                                         |
| [`__fixtures__/`](__fixtures__/)                 | Manually maintained test data                                                                                               |
| [`__tests__/`](__tests__/)                       | Mirrors `src/` with `.test.ts` suffix                                                                                       |

**GraphQL type safety**: Inline ``graphql(/* GraphQL */ `...`)`` operations in `src/**` and `bin/**` are typed via [@graphql-codegen/client-preset](graphql.config.ts). The custom client [src/common/github/api/client.ts](src/common/github/api/client.ts) wraps fetch with `TypedDocumentString` for compile-time validation. Preview features come from [schema/github/schema.graphql](schema/github/schema.graphql).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [andykenward/github-actions-cloudflare-pages](https://github.com/andykenward/github-actions-cloudflare-pages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
