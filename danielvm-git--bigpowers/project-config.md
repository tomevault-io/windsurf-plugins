---
trigger: always_on
description: \"Build editorial/magazine/report webpages on a GENUINE Müller-Brockmann modular grid (International Typographic Style) — not a decorative one. Encodes the discipline (columns + modules + baseline, grotesque type, flush-left, restrained black/white/red palette) AND the hard-won front-end engineering to make the grid real, visible, and verified: one CSS-variable source of truth, an interactive grid-toggle overlay that lives in the SAME content box as the content, subgrid \\\"bands\\\" so every el
---



# Müller-Brockmann Grid Systems — built real, visible, and verified

Josef Müller-Brockmann (1914–1996), Zurich; *Grid Systems in Graphic Design* (1981) is the corpus. The grid is treated as an ethic, not decoration: **"The grid system is an aid, not a guarantee. It permits a number of possible uses and each designer can look for a solution appropriate to his personal style. But one must learn how to use the grid; it is an art that requires practice."** This skill encodes that discipline AND — the part most attempts get wrong — the front-end engineering to make the grid genuinely load-bearing on the web, plus a harness that PROVES it.

> **HARD GATE** — Do NOT ship a grid page without running `verify_grid.js` first. A visually plausible layout is not evidence of grid adherence — the harness is. Zero failing assertions is the only passing bar.

> Two real review notes this skill exists to prevent:
> 1. *"the grid is just slapped on top and misaligned"* → the overlay wasn't in the same content box as the content (see §2.2).
> 2. *"the H in the headline is off the grid"* → the headline's BOX was on the grid but its INK wasn't; large glyphs carry a side-bearing (see §2.6). **Box-on-grid ≠ ink-on-grid.**


## PART 1 — THE DISCIPLINE (decide before drawing)
- **Objective order.** The grid brings "constructive thought," legibility, and "objective and functional" design. Restraint is the point; the system, not the ego, organizes the page.
- **Modular grid.** Divide the type area into a field of **modules** — columns AND rows — separated by consistent **gutters**, inside defined **margins**. Text and images occupy whole modules. Müller-Brockmann specimens common field counts (8 / 20 / 32 fields). For the web, a **12-column grid + 8px baseline** is a robust general default; a **6×6 or 4×8 modular field grid** when you want visible rows too.
- **Baseline grid.** Vertical rhythm is sacred: **leading = a whole multiple of the baseline unit**, and every element snaps to it. This is what makes facing columns and images line up across the page.
- **Typography.** A **grotesque sans** (Akzidenz-Grotesk / Helvetica; on the web Inter, Helvetica Now, Archivo). **Flush-left, ragged-right.** Few sizes, large jumps in **scale** for hierarchy; objective, not expressive. Big **numerals/data set large** is a signature move.
- **Palette.** Pure white paper, near-black ink, **one accent — red is canonical**. Avoid the warm-cream "Claude look"; **never blue/purple gradients** (hard house rule).
- **White space + asymmetry.** Generous margins; asymmetric compositions held in tension by the grid.


## PART 2 — MAKE THE GRID REAL ON THE WEB (the load-bearing engineering)
`grid_tokens.py` emits this whole scaffold correctly; the rules below are why it's built the way it is.

### 2.1 One source of truth
Put every grid parameter in `:root` CSS variables — `--cols, --gutter, --margin, --bl (baseline), --lh (leading=3×bl), --maxw`. **Content and the overlay both read these same variables.** Never hand-author the overlay separately or it will drift.

### 2.2 The overlay MUST live in the SAME content box as the content  ← #1 bug
Failure mode: content sits in a centered `max-width` container while the overlay is a **full-width sibling** of the section. On any viewport wider than `--maxw`, the centered content and the full-width overlay no longer share column positions → "slapped on top / misaligned."
**Fix:** put `.guides` *inside* the same `.wrap`, and draw the column guides with `left/right = var(--margin)` and the **same** `repeat(var(--cols),1fr)` + `column-gap:var(--gutter)`. Then the overlay columns **are** the content columns at every width. Add left/right margin lines at `var(--margin)`.

### 2.3 Place every element by column LINE via subgrid bands
Don't eyeball spans. Each horizontal **band** spans all columns and re-exposes them:
```css
.band{grid-column:1 / -1; display:grid; grid-template-columns:subgrid; column-gap:var(--gutter); align-items:start;}
@supports not (grid-template-columns:subgrid){ .band{grid-template-columns:repeat(var(--cols),1fr);} }
```
Children place with `grid-column: <startline> / <endline>` (e.g. `1 / 6`, `6 / 13`). Every headline, paragraph, photo, caption now snaps to identical lines.

### 2.4 Lock vertical rhythm to the baseline
- Leading = `--lh` (e.g. 24px = 3×8). **Every line-height a multiple of the baseline, in px (not unitless) for display type** — unitless line-heights on large type push the box off the grid.
- Every margin/padding a multiple of the baseline. Spread top/bottom padding a multiple too, so content starts on a line.
- **Media heights = multiples of the leading** (e.g. 240/360/432/480px) so a photo's top AND bottom both land on lines.
- Hairline rules sit inside a baseline-height band, not free-floating.

### 2.5 The toggle (sizzle within the sizzle)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danielvm-git/bigpowers](https://github.com/danielvm-git/bigpowers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
