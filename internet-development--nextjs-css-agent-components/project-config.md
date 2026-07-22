---
trigger: always_on
description: Operating manual for AI agents working in `nextjs-css-agent-components`. Read this end-to-end before editing.
---

# AGENTS.md

Operating manual for AI agents working in `nextjs-css-agent-components`. Read this end-to-end before editing.

## What this repo is

A Next.js 16 + React 19 + vanilla-CSS starter from Internet Development Studio. It powers `wireframes.internet.dev` and acts as the studio's design-system reference. Everything is intentionally vanilla — no Tailwind, no SASS, no PostCSS plugins, no CSS-in-JS, no UI library, no test runner, no state manager. Styles are CSS Modules using **native CSS nesting** (the `&` selector is plain CSS, compiled by Next.js's built-in Lightning CSS — no preprocessor required). The look is a precise, monospace-leaning, low-chrome wireframe aesthetic, and the precision is the product. Pixel values, type tokens, and theme variables are deliberate and should not be paraphrased away.

> The package was previously named `nextjs-sass-starter`. SASS has never been a dependency in the current codebase — all styling is vanilla CSS. The package is now `nextjs-css-agent-components`.

## Run / build

- `npm install`
- `npm run dev` — Next dev server on port `10000`
- `npm run build` — production build (TypeScript build errors are NOT ignored; see `next.config.ts`)
- `npm run start` — production server on port `10000`
- `npm run format` — Prettier write across `**/*.{tsx,ts,js,css,json,md}`
- `npm run format:check` — Prettier check (reports unformatted files without writing)

There is no test runner and no linter configured. The only build-time gate is `next build` (which typechecks). Don't add a test runner or ESLint config without being asked.

## Top-level layout

```
app/                    Minimal app-router scaffolding (head, layout, manifest, robots, sitemap)
pages/                  Pages router — this is where routes live (pages/_app.tsx, pages/examples/**)
pages/api/              API routes (currently `index.ts`, `aes.ts`)
pages/examples/         Live demo gallery for every component (animations, components, empty, features, fonts, system)
components/             App-level shell (DefaultLayout, Page, Providers, DefaultMetaTags, logos)
elements/               TIER 1 — atoms. Files import zero other in-repo components.
elements/icons/         Inline SVG icons (currentColor, props-spread). Includes social/.
elements/type/          Typography (H1..H5, Lead, P, Title, Text, SubText, …) + form typography
elements/controls/      Input, TextArea, Loader (form controls with no deps)
elements/marks/         Tag, ActionItem, SmallButton, SignatureBox (small inline document UI)
elements/layouts/       Page/region layout shells (Thin, Wide, Grid, TwoColumn*, Dashboard*, …)
elements/sections/      Full-height / half-height / horizontal section wrappers
elements/scroll/        Scroll-driven containers
elements/charts/        D3 chart components
elements/visuals/       FlippableTiltCard, Isometric*, ImageBlock, Video, ToolbarControlsFixed, …
elements/motion/        TextSwapper (React-driven motion)
elements/diagrams/      ArrowLine and friends
elements/shells/        DefaultLayout, DefaultMetaTags, AnyTextHeader, ListItem, the two logos
components/             TIER 2 — molecules. Compose tier-1 atoms. Flat, 12 components.
                        (Button, Checkbox, Select, Table, Footer, FormUpload, MonospacePreview,
                         BlockFade, FadeManager, CheckmarkItem, FormChangePassword, FormSettingsPrivacy)
patterns/               TIER 3 — organisms. Page-level units composed from atoms + molecules.
patterns/chrome/        Navigation, KeyHeader, HamburgerMenuButton, Page, Providers
patterns/modals/        All concrete modal components (ModalAuthentication, ModalNavigationV2, …)
patterns/demos/         All Demo*.tsx — gallery showcases used by pages/examples/
runtime/                Non-component infrastructure. Not in the tier system.
runtime/modals/         ModalContext + GlobalModalManager (provider + renderer)
runtime/detectors/      OutsideElementEvent listener
runtime/testing/        GridTestOverlay (dev-only visual grid)
common/                 utilities.ts, hooks.ts, constants.ts, queries.ts, server.ts (shared client/server helpers)
modules/                Lower-level modules (aes, cookies, cors, vary, object-assign)
public/                 Static assets (favicons, app icon, skills/)
global.css              Reset, @font-face for ServerMono, all CSS custom properties (colors, themes, type, fonts)
animations.css          Global @keyframes (blur, fade, slideUp/Down/Left/Right)
```

## Three-tier component hierarchy — read before adding/moving anything

The component model is **Elements → Components → Patterns** (the standard design-system vocabulary used by Polaris, Material, Carbon, Atlassian). Tier is determined by _deepest dependency_, not by where a component "feels like" it belongs.

| Tier | Folder        | Rule                                                                                   | Example                                                                                                  |
| ---- | ------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [internet-development/nextjs-css-agent-components](https://github.com/internet-development/nextjs-css-agent-components) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
