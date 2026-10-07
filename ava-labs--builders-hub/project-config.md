---
trigger: always_on
description: Instructions for coding agents and contributors in this repo. The repo is the source for [build.avax.network](https://build.avax.network): docs, Academy courses, the Builder Console, the Explorer, and the marketing pages.
---

# AGENTS.md

Instructions for coding agents and contributors in this repo. The repo is the source for [build.avax.network](https://build.avax.network): docs, Academy courses, the Builder Console, the Explorer, and the marketing pages.

## Next.js 16

This repo uses Next.js 16 with Turbopack. Its APIs and conventions can differ from what you know. Read the guide in `node_modules/next/dist/docs/` before you use a Next.js API.

- `proxy.ts` replaces `middleware.ts`, and it exports `proxy`.
- `next.config.mjs` sets `agentRules: false`, so `next dev` writes no agent files. This file holds the instructions.

## Stack

- Next.js 16, React 19, TypeScript 5.9, Tailwind CSS 4 (CSS-first config in `app/global.css`, no `tailwind.config`).
- fumadocs (`fumadocs-core`, `fumadocs-ui`, `fumadocs-mdx`) for MDX content.
- Prisma 6 on PostgreSQL (`prisma/schema.prisma`).
- wagmi 3, viem 2, `@avalanche-sdk/client` and `@avalabs/avalanchejs` for chain access.
- AI SDK 6 (`ai`, `@ai-sdk/anthropic`, `@ai-sdk/openai`).
- Package manager: yarn 1 (`packageManager` in `package.json`, `yarn.lock`). Do not use npm or pnpm in the repo root.

## Layout

| Path | Contents |
|---|---|
| `app/` | App Router. `(home)` holds the marketing pages, `explorer/[network]`, `stats`, `solutions`. Also `academy/`, `docs/`, `console/`, `api/` |
| `components/toolbox/` | The Builder Console: tools, flows, hooks, stores, `coreViem` |
| `components/console/` | Console shell: sidebar, header, command palette, `console-flows.ts` |
| `components/explorer-v2/` | The Explorer UI |
| `content/` | MDX: `docs/`, `academy/`, `blog/`, `integrations/` |
| `lib/` | Shared modules: `source.ts` (fumadocs loaders), `stats-api.ts`, `explorer-query/`, `pchain-rpc.ts` |
| `server/services/` | Server-side business logic |
| `prisma/` | Schema, migrations, seeds, client singleton |
| `scripts/` | Build, generation and check scripts |
| `tests/unit/` | Vitest unit tests |
| `tests/e2e/` | Browser and API tests (tester-army/e2e). A separate npm package |

## Commands

```bash
yarn install          # postinstall: prisma generate, fumadocs-mdx (.source/), generated files
yarn build:remote     # download remote MDX (API and RPC reference). Run once before yarn dev
yarn dev              # next dev --turbopack
npx tsc --noEmit      # typecheck (tests/ is excluded)
npx vitest run tests/unit/explorer   # one unit test folder; npx vitest run for all
yarn build            # tsc --noEmit && next build
```

The repo has no `lint` script. Run ESLint on the paths you changed, as CI does (see "Checks").

For the browser and API tests, read `tests/e2e/README.md`. They run against a running site:

```bash
cd tests/e2e && npm ci && npx playwright install chromium
E2E_BASE_URL=http://localhost:3000 npm test
```

## Checks

| Check | Runs on | What it does |
|---|---|---|
| `.github/workflows/console-ci.yml` | PRs that touch the Console, contracts or `content/academy` | tsc, toolbox ESLint, folder and import rules, `scripts/check-console-design.sh`, `scripts/check-academy-embeds.mts` |
| `.github/workflows/explorer-ci.yml` | PRs that touch the Explorer | tsc, Explorer ESLint |
| `.github/workflows/unit.yml` | Every PR, and each push to `master` | The whole Vitest suite (`npx vitest run`) |
| `.github/workflows/e2e.yml` | Every PR, each push to `master`, and nightly | Browser tests (desktop, phone and in-app browsers) and API tests. A PR runs the tests its changed files reach plus a smoke set (`tests/e2e/select`); `master` and the nightly run all of them |
| `.github/workflows/e2e-explore.yml` | Nightly, or by hand | AI bug hunt: one `e2e explore` per charter in `tests/e2e/explore/charters.json`, on production. Findings go to the job summary |
| `.github/workflows/commitlint.yml` | Every PR | Conventional Commits on every commit |
| `.husky/pre-commit` (lint-staged) | Each local commit | Prettier and ESLint on the toolbox, the design check on the Console, ESLint on other files that a block covers, then `tsc --noEmit` |
| `.husky/commit-msg` | Each local commit | commitlint |

## Rules that CI enforces

**Explorer** (`eslint.config.mjs`, the `EXPLORER` list):
- No unused code.
- Each shared helper has one home: `lib/explorer-query/values.ts`, `components/explorer-v2/format.ts`, `lib/explorer-query/edges.ts`. Import the helper; do not define a copy.
- A file has at most 600 code lines. A file over the cap has a ceiling in `scripts/explorer-size-ceilings.json`, and a ceiling can only go down. Split a file before you add to it.

**Console** (`eslint.config.mjs`, `console-ci.yml`, `scripts/check-console-design.sh`):
- Use `useResolvedWalletClient`, not wagmi `useWalletClient`.
- Contract hooks use `useContractActions`, not `viem/actions`.
- Use `@/` imports, not `../../../`.
- Use `zinc-*` classes, not `gray-*` or `slate-*`.
- Use `next/link`, not a raw `<a href>` (except `#` anchors and links with `target`).
- Do not wrap contract-hook calls in `notify()`: the hooks notify.
- Do not call `createPublicClient` in the toolbox. Use `usePublicClientForChain`, `makePublicClientForChain` or `useChainPublicClient`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ava-labs/builders-hub](https://github.com/ava-labs/builders-hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
