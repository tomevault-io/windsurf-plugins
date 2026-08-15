---
trigger: always_on
description: Working notes for this repo. Read this before changing anything under
---

# AGENTS.md

Working notes for this repo. Read this before changing anything under
`components/` — most of the code is one WebGL machine and a lot of it is
non-obvious in ways that look like bugs.

## What this is

A single-page portfolio carousel. Eighteen project cards sit on a ring that is
mostly off-screen to the left; you see an arc of it. Scroll, drag or swipe
turns the ring, and it snaps so a card faces front. The cards are not DOM
elements or textured quads — the whole ring is **one full-screen fragment
shader** drawing signed distance fields, which is what lets neighbouring cards
melt into each other ("goo") and string honey-like threads as they separate.

Everything visible is either that one shader pass or a handful of absolutely
positioned DOM labels over the top of it.

## Commands

```bash
npm run dev      # localhost:3000
npm run build    # also the fastest correctness check
npm run lint     # eslint
npx prettier --check "components/**/*.{js,jsx}" "app/**/*.{js,jsx}"
```

There are **no tests**. `npm run build` plus `npm run lint` is the whole safety
net. GLSL is compiled at runtime, not at build time, so a shader typo builds
fine and fails in the browser console — check shader edits by loading the page.

## Layout

```
app/
  page.js              renders <Carousel />, nothing else
  layout.js            root layout + metadata
  globals.css          Tailwind v4 import, @font-face, page background

components/
  Carousel.jsx        the component. renderer, resize/fit, input, spin
                       physics, the per-frame layout loop, the entry timeline
  ring/
    projects.js        the eighteen projects, in ring order
    params.js          every tunable, as a factory
    utils.js           TAU/DEG, easings, signedOffset, chase
    atlas.js           packs all art into one texture, incrementally
    meta.js            the two type lockups and their morph
    splitText.js       the intro heading ("Works '26")
    tag.js             the "View" tag that rides the cursor
    gui.js             lil-gui dev panel, dynamically imported
  shaders/
    planeShaders.js    the ring: SDFs, goo, glass lip, tag. ~430 lines of GLSL
    textShaders.js     the per-glyph reveal for the intro heading
```

`Carousel.jsx` is ~1400 lines and deliberately so. The fit logic, pointer
handling, layout loop and timeline share about twenty closure variables. They
have been left together because threading a context object through them reads
as tidier in a file tree and is harder to follow in an editor.

## The three coordinate ideas

Get these wrong and nothing else makes sense.

**World px.** Origin at screen centre, **Y up**. This is the space the shader
evaluates in, so pointer coordinates are converted into it once, on the way in,
and never again. Page Y is down, hence sign flips whenever the two meet.

**Ring slot vs plane index.** Planes are numbered in _fan order_ — the seed
first, then alternating either side of it, so index 0,1,2,3,4 sits at slot
0,+1,−1,+2,−2. `signedOffset(i)` converts. **Consecutive indices are on
opposite sides of the ring.** Anything derived from index rather than slot will
be subtly wrong; art used to be dealt by index and made the project column step
two names per slot.

**`g`, the stage scale.** Every plane-pixel measurement is multiplied by `g`,
so the ring resizes as one piece and the goo keeps its proportions. `g` folds
in both the entry's `endScale` and the window fit. If you add a measurement in
px, decide whether it goes through `g` — most do.

## Responsive model

Params are authored against a **reference window** (`refWidth: 1512`, a 14"
MacBook Pro at default scaling) and scaled by `fit = viewW / refWidth`, clamped
to `[minScale, maxScale]`. Width alone drives it by default, which keeps the
composition exactly self-similar: the ring's centre is placed as a fraction of
the viewport, so scaling its radius by the same fraction lands the front card
on the same relative spot at any width.

On top of that are two **bands**, computed in `refit()` and applied as
multipliers, not replacements:

|          | `narrowAt` ≤ 1024 | `tightAt` ≤ 640         |
| -------- | ----------------- | ----------------------- |
| plane    | ×1.25             | —                       |
| radius   | ×1.3              | ×0.82 (stacks → ×1.066) |
| text     | ×1.5              | name ×1.5 again         |
| posX     | −2.5              | −3.5                    |
| endScale | 4.22              | —                       |
| layout   | all four labels   | name only, bottom-right |

Two rules when touching this:

- **`refit()` runs on resize only.** The layout loop reads `fit`/`planeK`/
  `radiusK`/`textK` thousands of times a second and must not be recomputing
  them.
- **Band _flags_ are stored, not resolved values,** so anything picked off them
  (`posX`, `endScale`) still responds to the dev panel between resizes.

If you tune on a machine that isn't 1512 wide, set `refWidth` to your window
first — the **fit** folder has a button that reads it off the live one.
Otherwise you are tuning against a scale factor that isn't 1 and everything
will be wrong everywhere else.

## Non-obvious things that will bite you

**`uScale` is a packed vec4.** `xy` is the birth scale, `z` is brightness (for

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Yousuf-developer/Viscose-carousel](https://github.com/Yousuf-developer/Viscose-carousel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-15 -->
