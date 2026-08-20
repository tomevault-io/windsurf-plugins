---
trigger: always_on
description: This file provides guidance to Codex when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex when working with code in this repository.

## Commands

- `npm run dev` — start the Next.js dev server
- `npm run build` — production build
- `npm run lint` — ESLint (flat config, `eslint.config.mjs`)

There is no test suite.

## Code style

Do not add explanatory comments to the code. No comments justifying why a line is written a certain way, no comments restating what the code does, no JSDoc unless it already exists on the surrounding API. The code should stand on its own; put the explanation in the commit message or the chat response instead. Leave existing comments alone.

## Commits & PRs

Never append any footer, trailer, or attribution mentioning AI assistance in commit messages, PR descriptions, or anywhere else.

Commits are written entirely in English: a gitmoji title line (e.g. "✨ Add non-JS fallback content"), optionally followed by an English body explaining the change. Push directly to master — do not open pull requests.

## What this project is

An interactive portfolio (Next.js 16 App Router, React 19, TypeScript, Tailwind 4) whose entire desktop Hero/Details UI is rendered inside a WebGL canvas via React Three Fiber, while intentionally looking like a normal flat website. Phase 1 (flat WebGL UI) is done; Phase 2 (camera zoom-out revealing the "website" was a monitor in a 3D workstation scene) is planned — the R3F foundation exists to support that.

## Architecture

Three parallel render paths live in `src/app/page.tsx` (a single client component):

1. **Desktop (WebGL)** — `Scene.tsx` mounts an R3F `<Canvas>` (dynamically imported, `ssr: false`) fixed behind the page. `HeroText`, `Details`, and post-processing effects render *inside the canvas*; the DOM `<main>` is just a scroll spacer driving scroll-linked animation. Its height is `TIMELINE_VIEWPORTS + overflowViewports`, where the overflow is derived from the content — the page grows as copy is added.
2. **Mobile** (`useIsMobile`, <768px) — plain DOM components in `src/components/Mobile/`; the canvas still shows the model but skips hero text, details, and effects.
3. **No-JS fallback** — `src/components/NoJs/` renders real HTML content; the JS app lives under `.js-only-app` (toggled via CSS in `globals.css`).

Key mechanisms that span multiple files:

- **Scroll → 3D animation**: `HeroTransitionProvider` exposes three mutable refs (not state — updated per-frame without re-renders), read in `useFrame` by components inside the canvas. Scroll itself is smoothed by Lenis (`ReactLenis root`), mounted only after the loader finishes. See "Scroll timeline" below for what each ref means.
- **3D responsiveness**: `src/lib/heroSafeZone.ts` computes margins targeting a 16/9 safe zone; `HeroLayoutProvider`/`HeroLayoutContext` converts pixels to 3D units (`pxTo3DWidth/Height`) and exposes layout anchors (`leftX`, `rightX`, row Ys, `responsiveScale`). All in-canvas text positions itself from this context — never hardcode 3D positions for text.
- **Hybrid WebGL/DOM text**: visual text is rendered in WebGL, but synced real HTML elements are kept in the DOM for selection, accessibility, and SEO.
- **Tuning constants**: nearly every animation/layout magic number lives in `src/config/constants.ts` (`CONFIG`, `THEME`, `FONTS`). Put new tunables there, not inline.
- **Content**: all copy (hero, experience, projects, education, skills, bio) is in `src/data/content.ts`. Bio has three interchangeable variants (`bioVariants`), switchable from the Leva panel on `/debug`.
- **Model** (`src/components/Model.tsx`): Draco-compressed `.glb` from `public/glbs/`, `MeshTransmissionMaterial`, custom grab/drag/throw spring physics, idle rotation, and scroll-driven scale/position.
- **Post-processing**: `src/components/Effects/CustomAberration*` is a custom shader effect (chromatic aberration + grid distortion driven by mouse velocity), injected via `@react-three/postprocessing`. Desktop only.
- **Performance**: `<PerformanceMonitor>` in `Scene.tsx` dynamically adjusts canvas DPR (down to 0.75 below 45 FPS, up to 1.5 above 55 FPS).

## Scroll timeline

The desktop page is three stages, and the scroll length is derived from content, not fixed:

1. **Hero** — one viewport.
2. **Details** — Experience, Featured Projects, Education (left column) and Skills (right). Scrolls for as long as its content needs.
3. **Bio** — starts one full viewport below the point where stage 2 finishes scrolling, so it reads as its own screen. Only the sticky title remains from earlier stages.

`HeroTransitionProvider` publishes three refs:

- `progressRef` — 0→1 across the **hero→details transition only**, over an absolute distance of `innerHeight * (VIEWPORTS - 1)`.
- `detailsScrollRef` — pixels scrolled *beyond* that transition. The details group translates by this; that is what makes stages 2 and 3 scroll.
- `modelAnchorRef` — where the model should rest, as **viewport fractions**, published by `Details` and consumed by `Model`.

Two traps live here:

- **Do not collapse `progressRef` into one 0→N timeline.** Every model threshold (`INTERACTION_LOCK_EPSILON`, `SCALE_OUT_*`, `DETAILS_POPUP_*`) is expressed inside 0→1. Stretching the range silently retunes all of them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iTzRitual/folio-2026](https://github.com/iTzRitual/folio-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
