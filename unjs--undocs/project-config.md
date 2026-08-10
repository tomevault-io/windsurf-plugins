---
trigger: always_on
description: `undocs` — the UnJS documentation generator. A standalone **Nitro v3 + Vite +
---

# AGENTS.md

`undocs` — the UnJS documentation generator. A standalone **Nitro v3 + Vite +
Vue** app that renders a docs site from a directory of Markdown. It ships as a
CLI (`undocs dev` / `undocs build`) that a docs project depends on; the docs
project supplies only Markdown + one config file.

## Stack

- **Nitro v3** server — SSR-renders the app and serves the `/api/docs/*` content
  API. Preset auto-detected (`node-server` default, `vercel` on Vercel).
- **Vite** with two environments — `client` (browser bundle → `.output/public`)
  and `ssr` (`entry-server.ts`, whose `fetch` Nitro auto-wires as the renderer).
- **From-scratch router** (`src/app/router.ts`) instead of vue-router — a tiny
  reactive route + `AppLink`/`AppPage`/`AppLayout`.
- **`reka-ui`** (unstyled primitives) + **Tailwind v4** (`@tailwindcss/vite`).
- **`md4x`** (Markdown → comark AST) + **`rangi`** (highlight, synchronous),
  both server-only.

## Layout (`src/`)

### `src/app/**` — CLIENT (browser + SSR render)

- `main.ts` — client entry: seed stores from the payload → `createSSRApp` →
  hydrate `#root`. `entry-server.ts` — the SSR renderer.
- `router.ts` — `createAppRouter(history?)`, `useRoute`/`useRouter`,
  `createWebHistory`/`createMemoryHistory`. Static first-match route table with a
  mandatory trailing catch-all (docs page).
- `app.config.ts` — the **theme** defaults (`defineAppConfig`).
- `composables/` — `useAsyncData` (+`useLazyAsyncData`/`refreshAppData`), `useState`,
  `createError`, `useColorMode`, `useAppConfig`, `useRuntimeConfig` (client stub),
  plus content/query helpers (`useContent`, `useDocsNav`, `useDocsSearch`, …).
  `useHead`/`useSeoMeta` come from `@unhead/vue`.
- `components/app/` — the framework shims: `AppLink` (link + external `<a>`),
  `AppPage` (per-page `<Suspense>`, keyed by `route.path`), `AppLayout`
  (`route.meta.layout` resolver), `ClientOnly`. Other `components/` subdirs are UI.
- `content/` — `MarkdownRenderer.ts` (renders the comark AST to Vue via an
  explicit tag→component registry) + prose components.
- `layouts/`, `pages/`, `utils/`, `assets/`, `public/`.
- `ssr/` — the SSR state bridge: `server-context.ts` (per-request store via
  AsyncLocalStorage) and `payload.ts` (`window.__UNDOCS__` serialize/read).
- `inline/` — blocking `<head>` programs (asset recovery, color mode, embed
  theme). `.ts` source + the **committed compiled `.js`** rolldown emits;
  `entry-server.ts` inlines the artifact via `?raw`. Order in the shell matters:
  `color-mode` before `embed-theme` (an embedder's pinned mode must win). See
  `inline/README.md`.
- `env.d.ts` — types the `import.meta.{server,client,dev,prerender}` build flags
  and the `virtual:undocs/app-config` module.

### `src/server/**` — NITRO (node-only)

- `content/` — the content engine. `buildIndex()` (`builder.ts`) globs the docs
  dir, parses each file with `md4x`, runs block `transforms`, highlights with
  `rangi`, builds navigation/search/TOC/surround, and returns a `ContentIndex`. `store.ts` caches that index as a process singleton. `highlight.ts`, `icons.ts`,
  `types.ts`, `utils.ts` support it.
- `routes/api/docs/` — the content API (see below). Other routes: `raw/**` and
  `llms{,-full}.txt` (source Markdown for LLMs), `_og/**` (OG images via
  `takumi-js`), `_content` (build-stats debug page).
- `app-config.ts` — `generateAppConfig(docsDir)`: loads `.config/docs.*` via
  **c12**, renders landing markdown/hero code, returns the client app-config.
- Nitro modules/plugins: `bundle-docs.ts` (copies docs into the prod output),
  `vercel.ts` (Vercel output wiring), and dev-only live-reload
  (`dev-watch`/`dev-ws`/`dev-reload`).

### Root

`nitro.config.ts`, `vite.config.ts`, `cli/` (citty CLI), `schema/` (the docs
config JSON Schema + `.d.ts`), `docs/` (the repo's own docs, the default target),
`template/` (starter scaffold), `test/`.

## Commands

```bash
pnpm dev                       # vite dev on ./docs (:3000)
pnpm build                     # vite build → docs/.output/{public,server}
pnpm start                     # node docs/.output/server/index.mjs (:3000)
node cli/main.mjs dev  <dir>   # dev on an arbitrary docs dir (sets UNDOCS_DIR)
node cli/main.mjs build <dir>  # prod build → <dir>/.output
pnpm build:inline              # rolldown src/app/inline/*.ts → committed *.js
pnpm test                      # vitest run --coverage
pnpm typecheck                 # tsc --noEmit  (bare tsc: .vue imports don't resolve)
pnpm lint                      # oxlint && oxfmt --check   (fix: pnpm fmt)
```

The CLI sets `UNDOCS_DIR` (default `.`); `nitro.config.ts` and `vite.config.ts`
both read it, falling back to `./docs`. `pkgRoot` is resolved from
`import.meta.url` and used as Vite's `root`/`configFile`, so an installed bin
always loads OUR config, not the user's cwd. Because Vite's `root` is `pkgRoot`,
Nitro's `rootDir` is `pkgRoot` too — so its preset-derived output would land next
to undocs. The `rebaseOutput` nitro module (`src/server/rebase-output.ts`)
relocates that output tree onto `docsDir`, so the build always writes to
`<docsDir>/.output` (or the preset's equivalent). See the invariant below.

## The content API (the HTTP boundary)

Pages fetch content over HTTP — never by importing the engine. `useContent`'s

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unjs/undocs](https://github.com/unjs/undocs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
