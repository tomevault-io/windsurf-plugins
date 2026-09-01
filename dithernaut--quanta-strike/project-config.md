---
trigger: always_on
description: quanta-strike is a pixel font built from drawn pixel sheets. The rules below were
---

# quanta-strike — pixel font build pipeline

quanta-strike is a pixel font built from drawn pixel sheets. The rules below were
worked out carefully and are non-negotiable unless the user says otherwise. Read this
before doing anything.

## What it is
- A family of **strikes** — one design per target size: `quanta-strike-6`, `-10`, `-12`,
  `-14`, `-16`, `-18`, `-20` (more may exist). Each strike is its **own font family**
  named `quanta-strike-N`. The different SIZES are NOT weights/styles of one family
  (weights are — see below).
- Source art is a drawn pixel sheet: each strike is a PNG + JSON in
  `src/quanta-strike-N/<style>/`. `scripts/png-to-ttf.py` turns that pair into the TTF, so the TTF is a
  **build artifact** — the PNG + JSON are the only real source. (The TTF used to be
  exported by hand; that step is now scripted, and the build is self-contained.)
- **Weights ARE styles of one strike.** `<style>` is the weight folder: `regular/` is the
  one every strike has, and any sibling (`bold/`, `light/`, `semibold/`, `bold-italic/`)
  is another weight of that same strike family. Drop the folder in and the build picks it
  up — no registration anywhere. The sheet inside may spell the weight out or not
  (`bold/quanta-strike-12-bold.{png,json}` and `bold/quanta-strike-12.{png,json}` are the
  same sheet; `pick_sheet` in build.sh owns the precedence, most specific first, with a
  `-mono` variant sheet outranking either). The folder name is the only weight signal and must be a
  word from `WEIGHT_MAP` in `scripts/font-metadata-patcher.py`; an unrecognised one
  **fails the build** rather than silently shipping at weight 400 and colliding with
  regular. See [docs/SOURCE-FORMAT.md](docs/SOURCE-FORMAT.md#weights).
  - This is the ONE axis where strikes do behave like a normal family. Sizes are still
    separate families, and mono vs proportional are still separate families — only
    weights live inside a family, on one `font-family` with distinct `font-weight`s.
- **Two variants per strike, from the same source by default.** Every strike is built
  twice:
  - **mono** — `quanta-strike-N-mono`, the original fixed-advance behaviour
    (advance = `(glyph-width + glyph-spacing) × 128`). Used for coding/TUIs; this is
    the variant that gets Nerd Fonts. Its generation is unchanged, byte-for-byte.
  - **proportional** — `quanta-strike-N` (the base name), each glyph trimmed to its own
    ink (zero left side bearing, advance = `(ink-width + gap) × 128`). The gap is chosen
    per strike in precedence order: `--spacing V` (forces all strikes) → the strike
    JSON's `spacing` key → **`auto`**, which scales with strike size N (1px N<11, 2px
    11–18, 3px N>18; bigger strikes get more air). A `spacing` value is a pixel count or
    `"auto"`. More legible for body text; `glyph-width` is now conceptually the *max*
    width. No Nerd Fonts.
  Both are still their own font families (mono and proportional are NOT styles of one
  family) and both hold the pixel invariant below — trimming only removes whole empty
  pixel columns, so widths stay on the 128 grid and the cross-strike pixel is untouched.
  - **Optional dedicated mono sheet.** By default both variants build from the one
    `quanta-strike-N.{png,json}`. If a strike also has a `quanta-strike-N-mono.{png,json}`
    pair next to it, the **mono** variant is built from that sheet instead (the
    proportional variant always uses the plain one). This lets a strike ship a
    hand-tuned monospace design while keeping the shared source for everything else;
    with no `-mono` sheet, mono just uses the plain source. (Generally: a variant with
    a non-empty suffix uses `<family><suffix>.{png,json}` when both are present.)

## THE hard invariant (never break this)
- **1 pixel = 128 font units**, always. Glyph coordinates are multiples of 128.
- **em (unitsPerEm) = N × 128** for strike-N. So at font-size N px, one source pixel
  renders at exactly **1.0 CSS px**.
- Consequence: at each strike's **native size** (N px), the pixel is the SAME physical
  size on EVERY strike. This cross-strike pixel identity is the whole point of the
  family and must survive end-to-end (metadata, features, WOFF2).
- Rendered pixel = `font-size / N`, **independent of the units-per-pixel number**. So
  128 is arbitrary — do NOT try to "scale by changing units per pixel", it's a no-op.

## Authoring rules (how a strike is drawn)
- Canvas may be TALLER than N. The body (caps, x-height, descenders) lives in the
  bottom N pixels = the em; **accents/diacritics are drawn ABOVE the em and overshoot**
  (keeps caps full-size and accents legible without shrinking anything).
- Baseline = the bottom edge of the LAST filled pixel of capital `A`.
- Let `D` = pixel rows below the baseline (descender depth). Then:
  ```
  em      = N × 128
  descent = D × 128
  ascent  = (N − D) × 128     # = em − descent; this is the em TOP
  ```
  Draw caps to height `(N − D)` so their top lands on the em top; accents go above it.
- Only `D` is read from the drawing; caps/accents just fit under or overshoot the
  ascent line. Everything else follows from `N` and `D`.

## Build pipeline (`build.sh` orchestrates; scripts run via FontForge python)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dithernaut/quanta-strike](https://github.com/dithernaut/quanta-strike) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
