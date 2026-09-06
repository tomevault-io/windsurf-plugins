---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run build` — Rollup build → ESM, UMD, and minified UMD bundles in `dist/`
- `npm run dev` — Rollup watch mode (rebuilds on save)
- `npm run lint` — ESLint on `src/`
- `npm run format` — Prettier write on `src/**/*.{ts,css}`
- `npm run format:check` — Prettier check (CI-friendly)
- Demo: serve the repo root (e.g. `npx serve .`) and open `/demo/index.html`

## Code Structure

- `src/index.ts` — Entry point. Exports `Lightbox` class, auto-initializes on `[data-lightbox]` elements.
- `src/lightbox.ts` — Core `Lightbox` class: open/close morph, zoom, pan with momentum, preloading. All animation via rAF + spring physics.
- `src/physics/spring.ts` — Damped harmonic oscillator (`springStep`) using semi-implicit Euler integration. Spring presets and types.
- `src/style.css` — Overlay/backdrop/image styles, chrome UI (caption bar, close button, nav arrows, counter). Extracted to `dist/lightbox3.css` by PostCSS.

## Vocabulary

Use these terms consistently in code, comments, and conversation.

### Views (what the user sees)

- **closed** — No overlay, thumbnails only
- **opened** — Image visible, fit to viewport (`scale === 1`)
- **zoomed** — Image at native/larger scale (`scale > 1`), pannable
- **navigating** — Gallery mode, viewing one image in a multi-image set

### Transitions (animated movements between views)

- **opening** — closed → opened
- **closing** — opened → closed (FLIP morph back to thumbnail, or fade for text links)
- **dismissing** — opened → closed via vertical swipe (velocity-based commit/snap-back)
- **zooming in** — opened → zoomed
- **zooming out** — zoomed → opened
- **navigating** — opened → opened (next/prev image in gallery, strip slides laterally)

### Gestures (user-driven, in progress)

- **panning** — dragging while zoomed
- **momentum** — post-release glide (spring-driven)
- **snap-back** — rubber-band return to pan bounds
- **swiping** — horizontal swipe to navigate gallery (prev/next)
- **pinching** — two-finger pinch-to-zoom on touch devices
- **dismiss drag** — vertical drag to close (swipe-to-dismiss)
- **rubber-band bounce** — spring bounce at gallery edges when no more images

### Systems (internal engines)

- **spring engine** — rAF loop + `springStep()` that drives all animations
- **FLIP morph** — open/close technique (measure thumb → measure target → animate the delta)
- **preloader** — hover-triggered image prefetch
- **strip** — slide container (`lightbox3-strip`) holding current + adjacent slides for gallery navigation. Translated horizontally via spring for swipe/nav transitions.
- **chrome UI** — bottom caption bar (caption, counter, close button) + side nav arrows. Opacity animated via its own spring. Hidden when zoomed or single image.
- **dismiss gesture** — vertical swipe-to-dismiss system with velocity tracking, rubber-band resistance, and commit/snap-back threshold

## Animation Architecture

**Use `requestAnimationFrame` + spring physics for all animations and gestures.** Do not use the Web Animations API (WAAPI) or CSS transitions.

### Why

- **Single system**: One spring engine drives everything — open/close morph, zoom, pan momentum, snap-back. No mixing of animation paradigms.
- **Interruptible**: Any animation can be grabbed mid-flight. A user can catch an image during momentum, during a zoom transition, etc. rAF loops cancel cleanly; WAAPI `fill: forwards` creates layer priority conflicts with inline styles.
- **Velocity-aware**: Release velocity from gestures feeds directly into the spring's initial velocity. WAAPI has no concept of this — you'd need to pre-compute the entire curve.
- **Compositor-friendly**: All interactive animations use `transform` only (translate + scale), which stays on the GPU compositor thread.

### Interruptibility

**All animations must be interruptible.** A user must be able to grab, tap, or redirect any in-progress animation at any time. Never lock out input while an animation is running.

- Before starting any new animation, call `stopSpring()` to cancel the current rAF loop.
- Read the current spring state (position + velocity) as the starting point for the new animation — don't snap to a predefined start.
- Never use `isAnimating` as a gate that blocks user input during gestures. It should only prevent conflicting *programmatic* transitions (e.g. open during close), not block pointer events.

### Completion-only flags are traps

Never use a boolean that is only set on animation *completion* (e.g. `zoomed`) to gate user input. These flags create dead windows where clicks/taps are silently swallowed for the entire duration of the animation.

- **`zoomed`** is `true` only after zoom-in finishes and stays `true` until zoom-out finishes. During the zoom-out animation it is still `true`, so any check like `if (this.zoom.zoomed) return` blocks input for the entire transition.
- To check "is a zoom in effect right now (idle or animating)?" test the live spring state: `this.zoom.zoomed || this.zoom.scale !== 1`.
- Same principle applies to `isOpen` — set it when the action *begins* (overlay created), not when the animation lands.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lokesh/lightbox3](https://github.com/lokesh/lightbox3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
