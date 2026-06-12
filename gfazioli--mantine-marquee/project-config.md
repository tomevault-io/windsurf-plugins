---
trigger: always_on
description: `@gfazioli/mantine-marquee` — A Mantine 9 marquee component that creates seamless infinite-scrolling content loops with GPU-accelerated CSS keyframe animations, supporting horizontal/vertical directions, responsive props, and CSS-mask fade edges.
---

# CLAUDE.md

## Project
`@gfazioli/mantine-marquee` — A Mantine 9 marquee component that creates seamless infinite-scrolling content loops with GPU-accelerated CSS keyframe animations, supporting horizontal/vertical directions, responsive props, and CSS-mask fade edges.

## Commands
| Command | Purpose |
|---------|---------|
| `yarn build` | Build the npm package via Rollup |
| `yarn dev` | Start the Next.js docs dev server (port 9281) |
| `yarn test` | Full test suite (syncpack + oxfmt + typecheck + lint + jest) |
| `yarn jest` | Run only Jest unit tests |
| `yarn docgen` | Generate component API docs (docgen.json) |
| `yarn docs:build` | Build the Next.js docs site for production |
| `yarn docs:deploy` | Build and deploy docs to GitHub Pages |
| `yarn lint` | Run ESLint + Stylelint |
| `yarn format:write` | Format all files with oxfmt |
| `yarn storybook` | Start Storybook dev server |
| `yarn clean` | Remove build artifacts |
| `yarn release:patch` | Bump patch version and deploy docs |
| `diny yolo` | AI-assisted commit (stage all, generate message, commit + push) |

> **Important**: After changing the public API (props, types, exports), always run `yarn clean && yarn build` before `yarn test`, because `yarn docgen` needs the fresh build output.

## Architecture

### Workspace Layout
Yarn workspaces monorepo with two workspaces: `package/` (npm package) and `docs/` (Next.js 15 documentation site).

### Package Source (`package/src/`)
```
Marquee.tsx          # Component implementation (factory pattern)
Marquee.module.css   # CSS Modules with animation keyframes
Marquee.story.tsx    # Storybook stories
Marquee.test.tsx     # Jest unit test
index.ts             # Public exports
```

Single-component package — `Marquee` is the only exported component, built with Mantine's `factory<MarqueeFactory>` pattern (`useProps`, `useStyles`, `createVarsResolver`).

### Build Pipeline
Rollup bundles to dual ESM (`dist/esm/`) and CJS (`dist/cjs/`) with `'use client'` banner. CSS modules are hashed with `hash-css-selector` (prefix `me`). TypeScript declarations via `rollup-plugin-dts`. CSS is split into `styles.css` and `styles.layer.css` (layered version).

## Component Details

### Factory pattern
`Marquee` uses Mantine's `factory<MarqueeFactory>` which requires a `Factory` type declaring `props`, `ref`, `stylesNames`, and `vars`, plus `createVarsResolver` to map props to CSS custom properties on `.root`, `useProps` for default prop merging, and `useStyles` for the `getStyles` accessor.

### CSS custom properties split
CSS custom properties in `Marquee.module.css` control animation: `--marquee-duration`, `--marquee-gap`, `--marquee-animation-direction`, `--marquee-direction`, `--marquee-play-state`, `--marquee-fade-edge-size`. The `varsResolver` sets static props (`duration`, `reverse`, `fadeEdgesSize`). Three variables are set via inline `style` in `useStyles` because they depend on runtime state or hooks:
- `--marquee-play-state` — depends on hover state (`over`)
- `--marquee-direction` — `vertical` can be a responsive breakpoint object resolved by `useMatches`
- `--marquee-gap` — `gap` can be a responsive breakpoint object resolved by `useMatches`

The `varsResolver` only receives raw props and cannot call hooks, which is why these three are excluded from it and from `MarqueeCssVariables`.

### Animation mechanics
The marquee loop clones `children` into `repeat` wrapper `<div>`s (`.marqueeContent`) sharing the same CSS keyframe. Each clone translates by `translateX(calc(-100% - var(--marquee-gap)))`, exactly the distance to the next clone, making the loop geometrically seamless.

Key CSS decisions:
- `will-change: transform` on `.marqueeContent` / `.marqueeContentVertical` — promotes each clone to a GPU compositor layer, preventing frame drops.
- `backface-visibility: hidden` — prevents flickering on Safari/iOS during animation loop reset.
- `overflow: hidden` only on `.root`, not `.marqueeContainer` — having it on both creates an extra stacking context that interferes with GPU layer compositing.
- The CSS keyframe + `transform` approach runs entirely on the GPU compositor thread without touching layout or paint.

### Responsive `vertical` prop
`vertical` accepts `boolean | Partial<Record<MantineBreakpoint, boolean>>` (exported as `MarqueeVertical`). `useMatches` is always called (React hooks rules). A plain boolean is wrapped as `{ base: bool }` (no-op for `useMatches`). The resolved boolean is stored as `resolvedVertical` and used for `data-vertical`, class selection, and `--marquee-direction` inline style.

### Responsive `gap` prop
`gap` accepts `MantineSize | (string & {}) | Partial<Record<MantineBreakpoint, MantineSize | (string & {})>>` (exported as `MarqueeGap`). Same `useMatches` pattern as `vertical`. A plain string is wrapped as `{ base: gap }`. The resolved value passes through `getSize()` and is set as `--marquee-gap` via inline style.

### Fade edges — CSS mask system

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gfazioli/mantine-marquee](https://github.com/gfazioli/mantine-marquee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
