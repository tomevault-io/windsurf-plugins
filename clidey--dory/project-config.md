---
trigger: always_on
description: - Professional, neutral tone. Use exclamation points sparingly.
---

# CLAUDE.md

## Communication Style
- Professional, neutral tone. Use exclamation points sparingly.
- Clear, factual explanations without unnecessary enthusiasm.

## Development Requirements
1. Clean, readable code. Do not overengineer.
2. Do not remove or modify existing functionality unless justified.
3. Do not change existing variable names unless absolutely necessary.
4. Do not leave unused code lying around.
5. Ask questions to clarify the task before starting.
6. Use multiple subagents wherever possible for parallel work.

## Project Overview

**Dory** (`@clidey/dory` v0.33.0) is a static site generator for technical documentation, published on npm. Users install it as a dependency and run its CLI to build documentation sites from MDX files configured via a `dory.json`.

**Core stack:** Preact, Vite 6, TypeScript (strict), Tailwind CSS v4, MDX, SCSS

## Commands

```bash
pnpm install                  # Install deps (always pnpm, never npm/yarn)
pnpm run dev                  # Dev server with HMR
pnpm run build                # Full build: CLI + TypeScript + Vite
pnpm run build:cli            # Compile CLI only (tsconfig.bin.json → bin/dist/)
pnpm run build:docs           # Build docs site only (tsc + vite build)
pnpm run build:embed          # Build embed loader + widget
pnpm run preview              # Preview built site
pnpm run publish              # Build + publish to npm
```

**CLI commands** (after build):
```bash
node bin/dist/dory.js build           # Build a documentation site
node bin/dist/dory.js preview         # Serve built site (sirv, port 3000)
node bin/dist/dory.js verify:content  # Validate MDX compilation
node bin/dist/dory.js version         # Print version
```

**Commit style:** Conventional commits — `feat(scope):`, `fix(scope):`, `chore(scope):`

## Architecture

### Directory Structure
```
bin/                        CLI implementation
  dory.ts                     Main CLI entry (build, preview, verify:content, version)
  tsconfig.bin.json           Separate TS config for CLI compilation
docs/                       Documentation content (MDX files + dory.json)
  dory.json                   Site config: navigation, branding, colors, fonts, logo
k8s/                        Docker + Kubernetes deployment
  Dockerfile, dev.Dockerfile, docs.Dockerfile
  config/nginx-default.conf
scripts/                    Build automation scripts
  generate-frontmatter-json.js
  generate-llm-txt.js
src/
  main.tsx                  App entry point
  app.tsx                   Root component → renders Routes
  app.css, theme.scss       Global styles + CSS variables (--brand-primary, etc.)
  components/               Core UI components
    store.ts                  Global state: routes, navigation, frontmatter, search index
    routes.tsx                Routing logic (loads from docs/dory.json)
    search.tsx                Client-side search (FlexSearch + Algolia autocomplete)
    search-index.ts           Search index management
    mdx-provider.tsx          MDX component mapping
    navigation.tsx            Navigation tree
    hooks.tsx, dropdown.tsx, loading.tsx, notification.tsx
  ui/                       Layout components
    header.tsx, layout.tsx, mobile-navigation.tsx
    prev-next-link.tsx, table-of-content.tsx
  mdx/                      MDX rendering components (24 files)
    processor.ts              SINGLE SOURCE OF TRUTH for MDX preprocessing
    code.tsx                  Code highlighting (Prism)
    mermaid.tsx               Diagram rendering
    latex.tsx                 Math rendering (KaTeX)
    api-playground.tsx        Interactive API testing (OpenAPI)
    websocket-playground.tsx  WebSocket testing
    open-api.tsx              OpenAPI spec rendering
    async-api.tsx             AsyncAPI spec rendering
    accordion.tsx, callout.tsx, card.tsx, expandable.tsx
    fence.tsx, icon.tsx, list.tsx, source.tsx
    step.tsx, table.tsx, tag.tsx, text.tsx, mdx.tsx
  embed/                    Embeddable widget system
    loader.ts                 Script that injects widget iframe
    widget-iframe.ts          Widget iframe implementation
    types.ts                  Shared types
  sdk/                      Framework SDKs for embedding Dory
    index.ts                  Shared SDK logic
    react.tsx                 React SDK
    vue.ts                    Vue SDK
  plugins/                  Custom Vite plugins (11 files)
    (see Vite Plugins section below)
  config/                   MDX configuration (mdx.ts, mdx.js)
  utils/                    Utilities (colors.ts, fonts.ts, functions.ts)
  types/                    TypeScript type definitions (config.ts)
```

### Vite Plugins (`src/plugins/`)
The project uses **3 Vite config files**:
- `vite.config.ts` — Main app build (loads all plugins below)
- `vite.config.embed-loader.ts` — Builds `dist/embed.js` (IIFE)
- `vite.config.embed-widget.ts` — Builds `dist/embed-widget.js` (ES library)

Plugins loaded in main config (in order):
1. `html-files-middleware` — Intercepts HTML file requests in dev
2. `unified-mdx-plugin` — Preprocesses MDX content before compilation
3. `@preact/preset-vite` — Preact integration
4. `@mdx-js/rollup` — MDX compilation
5. `@tailwindcss/vite` — Tailwind CSS v4

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clidey/dory](https://github.com/clidey/dory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
