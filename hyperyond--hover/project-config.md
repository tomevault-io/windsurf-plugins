---
trigger: always_on
description: This file is the single source of truth for agents entering the Hover repository. Read this file first. It describes the current implementation and the boundaries agents must respect when working in it.
---

# Directory guide

This file is the single source of truth for agents entering the Hover repository. Read this file first. It describes the current implementation and the boundaries agents must respect when working in it.

## Core documentation index

- Product scope and onboarding: `README.md`.
- Architecture and protocols: this file (`CLAUDE.md`), `packages/core/README.md`.
- License: `LICENSE` (Apache-2.0).

## What Hover is

Hover is a Vite plugin (later: a Chrome extension) that injects a floating chat widget into the user's dev server page. The developer types natural-language instructions ("test the login flow"), an agent drives their *actual* Chrome via CDP + Playwright MCP, and the verified session can be one-click crystallized into a standard Playwright `.spec.ts` file under `__vibe_tests__/`.

The differentiator vs. Stagehand / Midscene / Playwright codegen is the **AI exploration → deterministic script** workflow: AI authors the test, but the saved artifact is plain `@playwright/test` code that runs in CI without an agent in the loop.

## Workspace directories

Workspace packages come from `pnpm-workspace.yaml`: `packages/*` and `examples/*`. The repo is pnpm + ESM throughout.

- `packages/core` is `@hover-dev/core` — the Node service. Owns agent invocation, Playwright CDP preflight, MCP config, and the WebSocket bridge between the injected UI and the agent process.
- `packages/widget-bootstrap` is `@hover-dev/widget-bootstrap` — host-agnostic helper that owns the four widget source files (`template.html`, `style.css`, `client.js`, `reducer.js`), the mtime-keyed read cache, and the script-bundle assembly. Exposes three layers (`getWidgetScript` / `buildWidgetBundle` / `readWidgetAssets`) so the Vite plugin and any future bundler plugin (webpack, Next, Astro) all emit a byte-identical widget.
- `packages/transform-source` is `@hover-dev/transform-source` — **private workspace package, never published to npm** (`private: true`). Owns the per-framework source-attribution transforms that stamp `data-hover-source="<rel-path>:<line>:<col>"` onto every host element in user code: `transformJsx` (Babel parser, covers React / Solid / Preact), `transformVue` (`@vue/compiler-sfc`, filters via `tagType === ELEMENT_TYPE_HOST` so PascalCase + kebab-case components are skipped), `transformSvelte` (`svelte/compiler`'s `parse({ modern: true })`, gates on `type === 'RegularElement'` so `Component` / `SvelteHead` / `TitleElement` are skipped), and `transformAstro` (`@astrojs/compiler`, async because the underlying parser is WASM-backed; filters via `type === 'element'` so PascalCase components AND kebab-case custom-elements are both skipped). All four report the `<` character's 1-indexed line + column for cross-framework consistency. **Distributed by inlining**: each of the 5 integration shims (vite/astro/nuxt/next/webpack) runs `tsup` with `noExternal: ['@hover-dev/transform-source']`, so the transform code lands inside the shim's own `dist/` and the published bundle has no bare `@hover-dev/transform-source` import. Consumers `pnpm add` only the shim. The transform-source npm-side deps (`@babel/*`, `@vue/compiler-sfc`, `svelte`, `@astrojs/compiler`, `magic-string`) get promoted into each shim's `dependencies` so they resolve normally at install time. Package is itself `main: dist/index.js` (dist-shape) because Node's strict ESM resolver can't follow `./types.js` imports back to on-disk `.ts` — same fix used by `@hover-dev/core` and `@hover-dev/widget-bootstrap`. Root postinstall builds it on fresh clone.
- `packages/vite-plugin` is `vite-plugin-hover` — the Vite plugin shim. Consumes `@hover-dev/widget-bootstrap` for the widget injection and `@hover-dev/core` for the service. Builds with `tsup` so it can `noExternal` the private `@hover-dev/transform-source` into its dist. The plugin's `transform` hook dispatches by extension: `.jsx`/`.tsx` → JSX, `.vue` → Vue SFC, `.svelte` → Svelte 5, `.astro` → Astro. `enforce: 'pre'` is load-bearing — it puts us before `@vitejs/plugin-react` / `vue` / `svelte` transforms, which would otherwise collapse JSX/templates into render-function calls and leave no host-tag AST to walk. Must be a no-op in production builds (`apply: 'serve'`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hyperyond/Hover](https://github.com/Hyperyond/Hover) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
