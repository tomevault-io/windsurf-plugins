---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**shadcn-react-table** (`@monabbir/shadcn-react-table`) is a shadcn/ui data table with **Material React Table (MRT V3) parity**, built on TanStack Table v8 and distributed as a **shadcn registry block** (consumers run `npx shadcn add` and the source is copied into their project). Workspaces:

- **`packages/shadcn-react-table`** (`@monabbir/shadcn-react-table`) — the product: the data-table module only. Depends on `@workspace/ui` for the shadcn primitives it renders.
- **`packages/ui`** (`@workspace/ui`) — the shared shadcn primitives (button, dialog, table, …), `lib/utils` (`cn`), and the single `globals.css`. This is the shadcn target — `pnpm dlx shadcn add` writes here. Consumed by both `@monabbir/shadcn-react-table` and `apps/web`.
- **`packages/eslint-config`**, **`packages/typescript-config`** — shared config presets.
- **`apps/web`** — the docs site, live examples, and the registry host that serves `/r/data-table.json`.

The data table is the product; everything in `apps/web` exists to document and ship it. The primitives in `@workspace/ui` are in-repo dependencies — they are *not* shipped to consumers (the registry declares them as `registryDependencies` so consumers get them from upstream shadcn in their own style).

## Critical: Next.js version

This repo uses **Next.js 16.2.6** (with React 19.2.4). Per `AGENTS.md`, this version has breaking changes vs. older Next.js — APIs, conventions, and file structure may differ from training data. Before writing any Next.js code, read the relevant guide in `node_modules/next/dist/docs/` and heed deprecation notices.

## Commands

Package manager is **pnpm 10.33.4** (Node ≥ 20). All top-level tasks are orchestrated by Turborepo and fan out via `dependsOn: ["^task"]`.

From repo root:
- `pnpm dev` — runs `next dev` for `apps/web` (persistent, uncached)
- `pnpm build` — builds all workspaces
- `pnpm lint` — ESLint across workspaces
- `pnpm typecheck` — `tsc --noEmit` across workspaces
- `pnpm format` — Prettier `--write` across workspaces

Per-package: the same scripts exist in each workspace's `package.json` and can be run directly, e.g. `pnpm --filter shadcn-react-table-web <script>`, `pnpm --filter @monabbir/shadcn-react-table <script>`, or `pnpm --filter @workspace/ui <script>`.

There is no test runner configured in this repo.

## Plans, reviews, and commits

- **Plans** live in `.ai/plans/<plan-name>-<date>.md` (repo root).
- **Code reviews** live in `.ai/reviews/<review-name>-<date>.md` (repo root).
- **Commits:**
  - Never commit unless explicitly asked.
  - Commits are authored as the repo user — never add a `Co-Authored-By` trailer.
  - Subject line only — no commit body/description.
  - Message format: `<prefix>: <msg>` (e.g. `feat: add column pinning`, `fix: debounce filter input`).

> Plans are stored under `.ai/plans/` and code reviews under `.ai/reviews/`.

## Adding shadcn/ui components

Run from the **repo root** (not inside `apps/web`):

```bash
pnpm dlx shadcn@latest add <component> -c apps/web
```

Components land in `packages/ui/src/components/` (the `@workspace/ui` package, not `apps/web`). The `apps/web/components.json` `ui` alias is wired to `@workspace/ui/components`, so the shadcn CLI writes there even when targeting the web app. Import with `import { Foo } from "@workspace/ui/components/foo"`.

shadcn config (both `components.json` files agree): `style: radix-sera`, `baseColor: neutral`, `iconLibrary: remixicon`, RSC + TSX enabled, CSS variables on. The single source-of-truth stylesheet is `packages/ui/src/styles/globals.css`.

## Architecture

Turborepo + pnpm workspaces. Workspaces are `apps/*` and `packages/*`.

### Consumption model: source, not built artifacts

Neither `@monabbir/shadcn-react-table` nor `@workspace/ui` has a **build step**. Their `exports` fields point directly at `./src/*` (`.ts`/`.tsx`), and `apps/web/next.config.ts` declares `transpilePackages: ["@monabbir/shadcn-react-table", "@workspace/ui"]` so Next compiles both packages' source in-tree. Consequences:
- Edits in `packages/shadcn-react-table/src/` and `packages/ui/src/` are picked up immediately by `next dev` — no rebuild needed.
- The packages cannot be consumed outside a bundler that transpiles them (intentional — they're private internal packages).
- `apps/web/tsconfig.json` maps `@monabbir/shadcn-react-table/*` → `../../packages/shadcn-react-table/src/*` and `@workspace/ui/*` → `../../packages/ui/src/*` so TS resolves the same source paths as runtime.

### The data-table module

The product lives at `packages/shadcn-react-table/src/components/data-table/`, organized by **layer**:

- `index.ts` — the public API barrel; the single entry consumers import (`@monabbir/shadcn-react-table/components/data-table`). Treat it as the API surface — keep it curated.
- `core/` — the engine and shared definitions: `data-table.tsx`, `use-data-table.ts`, `types.ts`, `constants.ts`, `config-context.tsx`, `icons.tsx`, `localization.ts`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Monabbir-Ahmmad/shadcn-react-table](https://github.com/Monabbir-Ahmmad/shadcn-react-table) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
