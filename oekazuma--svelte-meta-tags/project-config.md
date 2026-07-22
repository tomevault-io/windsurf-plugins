---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# Svelte Meta Tags Coding Agent Guide

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository layout

This is a **pnpm workspace monorepo**. The pnpm version is pinned via `packageManager` in the root `package.json`, and the Node version via `devEngines.runtime` in the root `package.json` — check that file for the current values. All shared dependency versions live in the `catalog:` block of `pnpm-workspace.yaml`; package `package.json` files reference them via `"catalog:"` and should not pin versions directly.

Workspace members:

- `packages/svelte-meta-tags/` — the published library (the only public package). Source is in `src/lib/`. The package consumes itself via the SvelteKit dev app under `src/routes/` for local iteration.
- `tests/svelte-5/` — a SvelteKit app dedicated to **Playwright e2e tests**. Each route under `src/routes/<feature>/` corresponds to a `tests/<feature>.test.ts` that asserts the rendered `<head>` markup. This is where new feature behavior must be verified (per `CONTRIBUTING.md`).
- `example/` — a runnable SvelteKit demo of the library; not part of the test pipeline.
- `docs/` — Blume (a markdown-first docs framework on Astro) documentation site (deployed to GitHub Pages by `.github/workflows/deploy-docs.yml` only when `docs/**` changes). It is **bilingual**: English pages live in `docs/content/`, Japanese translations in `docs/content/ja/` (mirrored paths). Site config (i18n, redirects, deployment base) lives in `docs/blume.config.ts`; section ordering comes from `meta.$.ts` files in the English content directories plus each page's `sidebar.order` frontmatter. When documenting a feature, update **both** locales.

Non-workspace directories worth knowing: `.agents/skills/` holds vendored Svelte AI skills (`svelte-code-writer`, `svelte-core-bestpractices`) pulled from `sveltejs/ai-tools` and pinned by hash in `skills-lock.json`; `.claude/skills/` symlinks into it. Do **not** hand-edit these files — they are managed by their lock file. Do follow their guidance when writing Svelte code.

## Common commands

Run from repo root unless noted. All `pnpm -r` scripts iterate workspaces.

```bash
pnpm install               # install workspace deps (uses pnpm catalog)
pnpm dev                   # run dev servers across workspaces
pnpm build                 # build all workspaces
pnpm package               # svelte-kit sync && svelte-package && publint (library output -> packages/svelte-meta-tags/dist)
pnpm check                 # svelte-check across workspaces
pnpm lint                  # prettier --check . && eslint .
pnpm format                # prettier --write .
pnpm test                  # runs every workspace's `test` (vitest in lib, playwright in tests/svelte-5)
```

Per-workspace commands (use these to scope work):

```bash
# Unit tests + benchmarks for deepMerge / define helpers
pnpm --filter svelte-meta-tags test
pnpm --filter svelte-meta-tags test:bench
pnpm --filter svelte-meta-tags exec vitest run tests/deepMerge/deepMerge.test.ts   # single file

# Playwright e2e (chromium / firefox / webkit). vite build && preview is started by playwright.config.ts.
pnpm --filter svelte-5 test
pnpm --filter svelte-5 exec playwright test tests/twitter.test.ts                  # single file
pnpm --filter svelte-5 exec playwright test --project=chromium                     # one browser
```

**`tests/svelte-5` imports `svelte-meta-tags` as `workspace:*` and resolves it from `dist/`.** Run `pnpm package` (or `pnpm --filter svelte-meta-tags package`) first whenever you change library source — CI does this before `pnpm build` and `pnpm test`. Without it, e2e tests will run against stale published artifacts.

## Library architecture

The public surface is intentionally small (`packages/svelte-meta-tags/src/lib/index.ts`):

- **`<MetaTags>`** — single Svelte 5 component (`MetaTags.svelte`) that renders **all** SEO/meta/link/Twitter/OpenGraph/Facebook tags into `<svelte:head>`. It uses runes (`$props`, `$derived`, `$effect`) and accepts `Partial<MetaTagsProps>`. Adding a new meta surface means: extend `types.d.ts`, render the conditional block in `MetaTags.svelte`, add a route under `tests/svelte-5/src/routes/<feature>/` and a corresponding `tests/<feature>.test.ts`, document it under `docs/content/` (en + ja), and add a changeset (see Releases).
- **Agent Skills** (`skills/svelte-meta-tags-setup/`, `skills/svelte-meta-tags-companion/`) — distributed via `npx skills add` (see `docs/content/guides/agent-skills.md`). If the canonical `deepMerge` + `define*` + `<MetaTags>` wiring pattern changes, or a new common usage mistake is identified, update the relevant `SKILL.md` alongside the docs change.
- **`<JsonLd>`** — renders `application/ld+json` either in `<svelte:head>` (`output="head"`, default) or inline (`output="body"`). The `schema` prop accepts `schema-dts` types, plain objects, arrays (multiple JSON-LD blocks), or a `{ '@graph': [...] }` wrapper. `@context: https://schema.org` is auto-injected. The literal `<script>` string is split (`'<scri' + 'pt'`) on purpose to bypass HTML parser confusion — preserve this when editing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oekazuma/svelte-meta-tags](https://github.com/oekazuma/svelte-meta-tags) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
