---
trigger: always_on
description: `vite-plugin-remix3` lets a Remix v3 app build through Vite + Nitro: client entries are bundled and hashed, SSR resolves URLs through Vite's manifest, and Nitro handles deployment. The plugin replaces what `createAssetServer` would do compile on demand in runtime.
---

# vite-plugin-remix3

`vite-plugin-remix3` lets a Remix v3 app build through Vite + Nitro: client entries are bundled and hashed, SSR resolves URLs through Vite's manifest, and Nitro handles deployment. The plugin replaces what `createAssetServer` would do compile on demand in runtime.

**Keep AGENTS.md updated with critical information about project.**

## Repo layout

| Path                                                       | Purpose                                                                                           |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------- |
| [src/index.ts](src/index.ts)                               | The plugin. Discovers client entries, registers `client` + `ssr` Vite environments.               |
| [build.config.ts](build.config.ts)                         | `obuild` config — bundles `src/index.ts` to `dist/index.mjs`.                                     |
| [package.json](package.json)                               | Publishes `dist/`. Peer-deps `vite ^8`. Built via `pnpm build`.                                   |
| [pnpm-workspace.yaml](pnpm-workspace.yaml)                 | Workspace root. Includes `./starter`. Overrides `vite-plugin-remix3` to `workspace:*`.            |
| [starter/](starter/)                                       | Example app consumed by `pnpm dev` / `pnpm dev:build`. Wired through the workspace override.      |
| [starter/vite.config.ts](starter/vite.config.ts)           | `plugins: [nitro(), remix()]`. No `server.ts` — Nitro auto-routes through the SSR env.            |
| [starter/app/router.ts](starter/app/router.ts)             | Route → controller mapping. Exports `router`. No default export.                                  |
| [starter/app/entry.server.ts](starter/app/entry.server.ts) | SSR entry (default `ssrEntry`). `export default { fetch: router.fetch }` — Nitro's fetch handler. |
| [starter/app/entry.client.ts](starter/app/entry.client.ts) | Client bootstrap. `import.meta.glob` map + `run({ loadModule, resolveFrame })` from `remix/ui`.   |

## Plugin shape

[src/index.ts](src/index.ts) exports a single `remix(options)` factory. Options:

- `entries` — globs (relative to root) for client entry inputs. Default `['app/entry.client.{ts,tsx,js,jsx,mjs}']`.
- `deny` — globs excluded from `entries` after expansion. Default `['app/**/*.server.*']`.
- `ssrEntry` — single SSR rollup input. Default `'app/entry.server.ts'`.

The plugin runs `enforce: 'pre'` and only implements `config()` — it returns an `environments` map registering `client` (`consumer: 'client'`, `manifest: true`, multi-input) and `ssr` (`consumer: 'server'`, single input). Entry discovery uses `node:fs/promises#glob` plus a tiny inline `globToRegex` for the deny list (no `picomatch` dep — swap in if patterns ever need brace expansion or character classes).

There is no `configureServer` middleware. Vite's default dev server serves source files at root-relative URLs; the framework's emitted URLs (`/assets/<source-path>`) are stripped of the `/assets` prefix client-side in [starter/app/entry.client.ts](starter/app/entry.client.ts) before dispatching through `import.meta.glob`. (An earlier PoC stripped this server-side via middleware; doing it in the client glob handler is enough.)

## How URLs work

In SSR-rendered HTML, scripts/styles come from `?assets=client` imports:

```ts
import entryAssets from "../entry.client.ts?assets=client";
// entryAssets.entry: "/assets/app/entry.client-<hash>.js" in prod, "/app/entry.client.ts" in dev
// entryAssets.js, .css: dependent chunks
```

Nitro's `?assets` resolver replaces the import with a manifest lookup at build time and emits `__fullstack_assets_manifest.js` next to the importer's bundle. In prod the SSR bundle inlines/imports it; the URLs hit static files Nitro serves from `.output/public/`.

The framework also emits URLs for islands (the `rmx-data` JSON) using `routes.assets.href({ path })` → `/assets/<source-path>`. In prod those resolve to manifest-mapped chunks via `import.meta.glob`; in dev they hit Vite's source pipeline directly.

## Build flow (prod)

1. Vite builds the **client** env → `.output/public/assets/<chunk>-<hash>.js` plus `.vite/manifest.json`.
2. Vite builds the **ssr** env → `node_modules/.nitro/vite/services/ssr/index.js`. Any `?assets=client` imports in this graph register their metadata.
3. Nitro fires `writeAssetsManifest()` — emits `__fullstack_assets_manifest.js` to each importer env's outDir using the data registered so far.
4. Vite builds the **nitro** env → `.output/server/index.mjs`. Nitro's auto-wiring sets the renderer to `ssrRenderer` (which dispatches to the ssr env at runtime).
5. Nitro copies public assets, writes route rules, finalizes `.output/`.

## Why an `ssr` env

Nitro's build order (see `nitro/dist/vite.mjs` `buildEnvironments`):

```text
build(client) → ... → writeAssetsManifest() → build(nitro)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pi0/vite-plugin-remix3](https://github.com/pi0/vite-plugin-remix3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
