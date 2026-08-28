---
trigger: always_on
description: This repo is a Vue 3 port of animated Hugeicons. **Your primary job, when
---

# Agent notes

This repo is a Vue 3 port of animated Hugeicons. **Your primary job, when
asked to continue icon work, is to author unique hover gestures** — one icon
at a time, from name, SVG shape, direction, and context — modeled on the 165
handcrafted originals. Do not invent a shared pulse or spin recipe.

- Icon source of truth: `packages/vue/src/icons/*.vue`
- Do not hand-edit generated registry files; run `pnpm registry:build`
- To re-port from the React repo: clone it, set `HUGEICONS_SRC`, run `pnpm icons:convert`
- Landing animations live in `apps/web` (Lenis + GSAP). Icon playback uses `motion-v`.
- Talk to the user in French. Keep code, comments, verbs, and this file in English.

When the task is authoring, converting, or reviewing icon motion, follow this
file end to end and load the project skill `.cursor/skills/author-icon-motion`.

## Current coverage — stopped after gallery page 10

The gallery paginates **80 icons** (`apps/web/src/components/IconGallery.vue`).
`scripts/dump-geometry.mjs` uses the same alphabetical order as the icon files
(`a.vue` vs `b.vue` string sort), so index `80` here is gallery page 2.

**Authored (unique choreography) exists only for gallery pages 1–10**, plus
`border-right-01` and `border-right-02` (start of page 11, authored with page 10).
Icons from page 11 onward still carry the generated marker and a name/shape recipe
from `scripts/icon-motion.mjs`. Do not claim the whole library is hand-authored.

| Gallery page | Indices | Range | Specs | File |
| --- | --- | --- | --- | --- |
| 1 | 0–79 | `a-arrow-down` → `ai-eraser` | 78 | `scripts/authored/page-01.mjs` |
| 2 | 80–159 | `ai-file` → `album-not-found-01` | 78 | `scripts/authored/page-02.mjs` |
| 3 | 160–239 | `album-not-found-02` → `analytics-up` | 79 | `scripts/authored/page-03.mjs` |
| 4 | 240–319 | `analytics` → `arrow-down-narrow-wide` | 77 | `scripts/authored/page-04.mjs` |
| 5 | 320–399 | `arrow-down-one-zero` → `arrow-vertical` | 74 | `scripts/authored/page-05.mjs` |
| 6 | 400–479 | `arrows-up-from-line` → `backpack-02` | 79 | `scripts/authored/page-06.mjs` |
| 7 | 480–559 | `backpack-03` → `bean` | 79 | `scripts/authored/page-07.mjs` |
| 8 | 560–639 | `beater` → `bitcoin-money-02` | 80 | `scripts/authored/page-08.mjs` |
| 9 | 640–719 | `bitcoin-money` → `book-04` | 79 | `scripts/authored/page-09.mjs` |
| 10 | 720–801 | `book-a` → `border-right-02` | 81 | `scripts/authored/page-10.mjs` |
| **11 (next)** | **802–879** | **`border-right` → `bubble-chat-preview`** | | **`scripts/authored/page-11.mjs`** |

Totals after page 10: **784 authored specs**, **6,122 icons**, **950** files
without the generated marker (165 originals + `accident` + 784 authored).

Resume at page 11 (`border-right`). Skip `bookmark-01` already done (handcrafted,
left out of page 10). After each page, stop and ask whether to continue or let
the user hover-review — do not silently walk the rest of the library.

## Three tiers of icon motion

An icon file falls into exactly one tier, identified by its header comment:

| Tier | Marker | Owner |
| --- | --- | --- |
| Handcrafted | neither marker | a human; **never regenerate** |
| Authored | `authored from scripts/authored` | `pnpm icons:author` |
| Generated | `generated from @hugeicons/core-free-icons` | `pnpm icons:generate-missing` |

`scripts/generate-missing-icons.mjs` only rewrites the generated marker, so
handcrafted and authored files survive a full regeneration.

The authoring emitter overwrites **generated or authored** files. It refuses a
handcrafted file unless the spec sets `rework: true`. Never set `rework` unless
the user explicitly asks to replace that original.

### Protected originals in pages 1–10 (do not touch)

Page 1: `add-circle`, `accident` (collision verb only; no authored/generated marker).
Page 2: `airplane-take-off-01`, `alarm-clock`.
Page 3: `alert-circle` (circle scale, stem stretch, dot bounce, duration 0.56).
Page 4: `archive-02`, `arrow-down-02`, `arrow-down-left-01`.
Page 5: `arrow-down-right-01`, `arrow-left-02` (translateX −2.6, scaleY 0.94),
`arrow-right-02`, `arrow-up-02` (translateY −2.6, scaleX 0.94),
`arrow-up-left-01`, `arrow-up-right-01` (translate 2.1, −2.1).
Page 6: `attachment`.
Page 7: `battery-charging-01`.
Page 8: none.
Page 9: `bluetooth`.
Page 10: `bookmark-01`.

Handcrafted arrows often animate a wrapping `<g>`. Sampling only `<path>` in
DevTools can show `moved: 0` even when the gesture plays.

## Authoring one gallery page

1. Dump geometry — author against those numbers, not guesses:

   ```bash
   node scripts/dump-geometry.mjs 802 80
   node scripts/dump-geometry.mjs airplane-mode-off
   ```

2. List every kebab name in the slice. For each file, read the header:
   skip if it has **neither** marker (handcrafted). Do not copy a skipped
   original into the spec file.

3. Write `scripts/authored/page-NN.mjs`. Export a default object keyed by
   kebab name. `parts` is positional: one entry per SVG primitive in source
   order; `null` leaves that primitive static. Shared easings live in
   `scripts/authored/_kit.mjs` (files prefixed `_` are skipped by the emitter).

4. Import kit helpers instead of duplicating them. Page-local helpers are fine

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mickaelrebeau/hugeicons-animated-vue](https://github.com/mickaelrebeau/hugeicons-animated-vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
