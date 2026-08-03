---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Status

Publishing is **enabled and CI-gated**: `release.yml` runs the full `lint`/`typecheck`/`test`/`build` verify job (reusing `ci.yml`) before `changeset publish`, so a release only goes out if the checks pass. The repo is green at the root and CI enforces that on every PR. Two packages are published to npm: `svelte-docsmith` (the library) and `create-svelte-docsmith` (the scaffolding CLI). Both are pre-1.0, so minor releases may carry breaking changes until v1.0.

## What this is

Svelte DocSmith is a framework/library for building documentation sites with Svelte 5 + SvelteKit. Components are based on shadcn-svelte (style: new-york) with themes from tweakcn. This is a pnpm/turbo monorepo with three workspaces:

- `packages/svelte-docsmith` — the publishable library (npm package `svelte-docsmith`). Source of truth for all reusable components/utilities (TOC, markdown renderers, shadcn UI primitives, etc.), exported via `src/lib/index.ts`.
- `packages/create-svelte-docsmith` — the scaffolding CLI (npm package `create-svelte-docsmith`). `npm create svelte-docsmith@latest` scaffolds a SvelteKit + Tailwind v4 project wired with DocSmith (preprocessor, Vite plugin, `DocsShell` layout, sample pages, llms.txt/sitemap endpoints). Plain Node + `@clack/prompts`, no build step; the starter lives in `template/` and pins a `svelte-docsmith` version that must be bumped when the library's template-facing surface changes.
- `sites/docs` — the documentation site that consumes `svelte-docsmith` as a `workspace:*` dependency. This is the dogfooding/reference app and where the markdown doc pages live.

## Commands

All commands are run from the repo root via turbo and fan out to both workspaces (`svelte-docsmith` and `docs`), unless scoped with `--filter`.

```bash
pnpm dev                 # dev servers for both workspaces
pnpm build               # build both (svelte-package for the lib, vite build for docs)
pnpm test                # vitest run (docs site has no tests; only the lib runs real tests)
pnpm test:coverage       # vitest run --coverage
pnpm typecheck           # svelte-check in both workspaces
pnpm check               # svelte-kit sync && svelte-check
pnpm lint                # prettier --check
pnpm format              # prettier --write
```

Run a command against a single workspace with turbo's filter flag, e.g.:

```bash
pnpm build --filter=svelte-docsmith
turbo run test --filter=svelte-docsmith
```

Or `cd` into the package and use its local scripts directly (e.g. `cd packages/svelte-docsmith && pnpm test`).

### Running a single test

Tests live only in `packages/svelte-docsmith` (vitest). Run a single file or pattern from that package directory:

```bash
cd packages/svelte-docsmith
pnpm vitest run src/lib/toc/from-content.svelte.test.ts
pnpm vitest run -t "some test name"
```

Vitest is configured with two workspace projects (`vite.config.ts`):

- `client` — jsdom environment, matches `src/**/*.svelte.{test,spec}.ts`, uses `@testing-library/svelte`
- `server` — node environment, matches other `src/**/*.{test,spec}.ts` files

## Architecture

### `packages/svelte-docsmith` (the library)

`src/lib` is organized by concern, not flat. The public contract is `src/lib/index.ts` plus the package `exports` subpaths; everything else is internal and free to move. Because no external consumer imports internal paths (they only use `svelte-docsmith`, `/vite`, `/preprocess`, `/content`, `/search`, `/llms`), the layout below can change without a consumer-facing break, as long as `index.ts` and the export map stay stable.

- `src/lib/core/` — the domain layer: framework-agnostic contracts and pure logic, no runtime or build deps. `config.ts` (`DocsmithConfig`, `defineConfig`), `content.ts` (the generated-index record types `DocsContentItem`/`SearchDoc`/`LlmsDoc`), `nav.ts` (`navFromContent`), re-exported through `core/index.ts`.
- `src/lib/buildtime/` — Node-only build tooling. `preprocess.ts` (the `svelte-docsmith/preprocess` mdsvex + Shiki preprocessor), `vite/` (the `svelte-docsmith/vite` plugin, split into `pages.ts`/`frontmatter.ts`/`extract.ts`/`collect.ts`/`git.ts` collaborators behind `vite/index.ts`), `highlight.ts` (shared Shiki config), and `markdown-layout.svelte` (the injected mdsvex default layout). Named `buildtime`, not `build`, because a `build/` dir is gitignored.
- `src/lib/generate/` — framework-agnostic string generators wired into routes: `sitemap.ts` and `llms.ts` (`generateLlmsTxt`/`generateLlmsFullTxt`).
- `src/lib/fallbacks/` — the checked-in stand-ins for the generated virtual modules (`content`/`search`/`llms`) that throw a clear error (shared `missing-plugin.ts`) when the Vite plugin is absent. The export map points `svelte-docsmith/content` and friends here.
- `src/lib/search/` — the client search engine (`create-search.ts`, `context.svelte.ts`, `snippet.ts`).
- `src/lib/toc/` — table-of-contents engine (DOM scanning + `IntersectionObserver` visibility, reactive state via `.svelte.ts` runes files). `toc.svelte.ts` is the main store; `from-content.ts` builds the server-rendered list.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geodask/svelte-docsmith](https://github.com/geodask/svelte-docsmith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
