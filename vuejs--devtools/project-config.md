---
trigger: always_on
description: Vue DevTools v9 ships two hosts over one kit and one client. The Vite plugin is a dock inside Vite DevTools. The Chromium extension is its own DevTools panel and does not go through Vite.
---

# AGENTS GUIDE

## Positioning

Vue DevTools v9 ships two hosts over one kit and one client. The Vite plugin is a dock inside Vite DevTools. The Chromium extension is its own DevTools panel and does not go through Vite.

- **`@vue/devtools-kit` + `@vue/devtools-client`** — shared runtime and UI: component tree, state, timeline, Pinia, pages, reactivity graph, and the v6 plugin setup API (`setupDevToolsPlugin`, alias `setupDevtoolsPlugin`).
- **`vite-plugin-vue-devtools`** — Vite host. Requires Vite 8.3.0+ and a project-level `@vitejs/devtools` (`devtools: { apply: 'serve' }` is recommended) next to `plugins: [vueDevTools()]`. This package does not register `DevTools()` and does not brand the shared Vite shell. Sibling repo: `../vite-devtools`. Read its [AGENTS.md](../vite-devtools/AGENTS.md) before changing dock, command, or open-in-editor behavior.
- **`@vue/devtools-chrome`** — Chromium MV3 host. Content backend calls `createDevtoolsKit({ clientName: 'chrome-extension' })` on the page and bridges the panel on channel `vue-devtools:chrome-extension`. It works on any page that runs Vue, including apps that never load Vite DevTools.

`devframe` / `@devframes/hub` (docs: [devfra.me](https://devfra.me)) matter on the Vite path: docks, commands, terminals, messages. The extension does not use that hub. Features that only exist when several Vite tools share a UI (custom tabs, commands, split screen, the floating inspector button) belong in Vite DevTools. Vue-level `addCustomTab` / `addCustomCommand` stay removed on both hosts. Custom inspectors, component hooks, timeline layers, and plugin settings stay on the Vue plugin API.

## Stack & Structure

Monorepo (`pnpm` workspaces). ESM TypeScript. Library packages bundle with tsdown. Node **22.12+**, pnpm **12.5.1** (`packageManager`). Versions live in `pnpm-workspace.yaml` `catalog:` (and matching `overrides`); package entries use `"catalog:"`.

### Packages

| Package                 | npm                        | Description                                                                                                                                                  |
| ----------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `packages/kit`          | `@vue/devtools-kit`        | Runtime, codec, RPC, plugin registry. Exports only `dist`.                                                                                                   |
| `packages/devtools-api` | `@vue/devtools-api`        | Public plugin API surface over the kit.                                                                                                                      |
| `packages/client`       | `@vue/devtools-client`     | Vue 3 + UnoCSS UI shared by both hosts. Private. Vite plugin copies the build into `packages/vite/client`; the extension loads the same client in its panel. |
| `packages/vite`         | `vite-plugin-vue-devtools` | Vite host. Serves the static client and registers an iframe dock with `@vitejs/devtools`.                                                                    |
| `packages/chrome`       | `@vue/devtools-chrome`     | Chromium MV3 host (background, content backend, DevTools panel). Private. Zip via `pnpm zip:chrome`.                                                         |

Other top-level directories:

- `docs/` — VitePress. Migration guide is `docs/guide/migration.md`.
- `playground/basic` — `@vue/devtools-playground-basic`.
- `playground/reactivity-graph` — graph tracer playground.
- `tests/` — unit, integration, performance, and smoke (Vite + Chrome).

```mermaid
flowchart TD
  kit["@vue/devtools-kit"] --> api["@vue/devtools-api"]
  kit --> client["@vue/devtools-client"]
  client --> plugin["vite-plugin-vue-devtools"]
  client --> chrome["chrome extension"]
  kit --> plugin
  kit --> chrome
  plugin --> viteHost["@vitejs/devtools"]
  viteHost --> hub["@devframes/hub"]
  hub --> devframe
```

## Dep Boundary

On the Vite host, docks, commands, terminals, and open-in-editor stay in `@vitejs/devtools`. Open-in-editor is forwarded as `vite:core:open-in-editor`, gated by `capabilities.openInEditor`. The extension panel does not offer open-in-editor; that action exists only in the Vite dock.

`vueDevTools()` options are `enabled` and `appendTo`. Dock layout, built-in integrations, and open-in-editor stay on Vite's `devtools` config. Runtime `budget` is a kit option.

## Architecture

- **Mount paths.** The Vue client is `{base}__devtools__/`. The Vite DevTools hub is `/__devtools/`. Those are different endpoints.
- **RPC.** The extension uses `vue-devtools:chrome-extension`. The Vite dock keeps the default `vue-devtools` channel so both can be installed together.
- **State.** Component state lives in `@vue/devtools-kit`. The client only renders it. `isComputedRef` has one implementation, in `packages/kit/src/codec`. A second copy in `state.ts` breaks the client build.

## Usage

### Vite plugin


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vuejs/devtools](https://github.com/vuejs/devtools) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
