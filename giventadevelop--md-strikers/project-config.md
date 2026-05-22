---
trigger: always_on
description: Hero section mobile image containment using aspect-ratio matching — the definitive technique for containing images of varying dimensions (portrait, landscape, ultra-wide) inside responsive containers without cropping or gaps
---


# Hero Section Mobile: Aspect-Ratio Image Containment (March 2026)

## Overview

The homepage hero section (`src/components/HeroSection.tsx`) has a 4-section split layout. Each section contains an image with **different dimensions and aspect ratios**. On mobile (`@media (max-width: 767px)`), we need every image fully visible (zero crop) AND filling its container (zero gaps). This rule documents the **aspect-ratio matching technique** that solved this across all three image sections.

All mobile overrides live in `src/app/globals.css` inside the `@media (max-width: 767px)` block.

## The Problem

Each hero section image has a different aspect ratio:

| Section | Image | Recommended Size | Aspect Ratio | Shape |
|---------|-------|-----------------|--------------|-------|
| 1 | Kerala portrait | 1000 x 1200 px | **5:6** | Tall portrait |
| 2 | Event slideshow | 2000 x 800 px | **5:2** | Wide landscape |
| 3 | Unite India banner | 1000 x 200 px | **5:1** | Ultra-wide strip |

Naive approaches fail:
- **`object-cover`** fills the container but **crops** the image (top/bottom or left/right)
- **`object-contain`** shows the full image but leaves **letterbox gaps** (purple/background bars)
- **Fixed `height` / `min-height`** either crops tall images or leaves empty space below short ones

## The Core Technique: Aspect-Ratio Matching

**The solution**: Set the CSS `aspect-ratio` on the container to match the image's native proportions. When container and image have the same ratio, `object-contain` fills the container exactly — zero crop AND zero gaps.

```
Container aspect-ratio  ===  Image aspect-ratio
         +
    object-contain
         =
  Zero crop + Zero gaps
```

### Why This Works

- `aspect-ratio: 5 / 2` on a `width: 100%` container → browser auto-calculates height proportionally
- Image with `object-contain` scales to fit → since ratios match, it fills 100% of the container
- `height: auto` lets the container shrink/grow with viewport width → fully responsive
- `background: #1a0a2e` (dark purple matching hero bg) hides any sub-pixel rounding gaps

## Section-by-Section Implementation

### Section 1 — Kerala Portrait (5:6, `object-contain` + `max-height` cap)

Section 1 is a **tall portrait** image. Using `aspect-ratio: 5/6` on mobile would make the container extremely tall (taller than the viewport). Instead, we use `object-contain` with a `max-height` viewport cap so the image is fully visible but doesn't dominate the screen.

**Why not `aspect-ratio` here**: A 5:6 portrait on a 390px-wide phone → 468px tall container, which is too much. The `max-height: 22vh` cap keeps it proportional.

**Why not `object-cover`**: Cover fills width but crops top/bottom. The Kerala image has text/content at the edges that gets cut off.

```css
@media (max-width: 767px) {
  /* Container: full-bleed, no decorations */
  .hero-left-panel {
    width: 100%;
    flex: 0 0 auto;
    min-height: 0;
    padding: 0;
    margin: 0;
  }

  .hero-left-image {
    width: 100%;
    max-width: 100%;
    height: auto;
    min-height: 0;
    aspect-ratio: auto;       /* NOT locked — height driven by max-height cap */
    padding: 0;
    border-radius: 0;
    box-shadow: none;
    border: none;
    background: #1a0a2e;      /* Blends side letterbox bars with hero bg */
  }

  /* Kill decorative pseudo-elements */
  .hero-left-image::before,
  .hero-left-image::after {
    display: none;
  }

  .hero-left-image img {
    border-radius: 0 !important;
    object-fit: contain !important;          /* Full image — no crop */
    object-position: center center !important;
    width: 100% !important;
    height: auto !important;
    max-height: 22vh !important;             /* Cap prevents portrait from being too tall */
    display: block !important;
  }
}
```

**Trade-off**: `object-contain` on a portrait image in a landscape container creates **side gaps**. The `background: #1a0a2e` makes these gaps blend with the hero's dark purple background, making them invisible.

**Tuning**: Increase `max-height` to show more of the image (e.g., `28vh`), decrease to show less (e.g., `18vh`).

---

### Section 2 — Event Slideshow (5:2, `aspect-ratio` match)

Section 2 is a **wide landscape** image. This is the canonical use of the aspect-ratio technique.

```css
@media (max-width: 767px) {
  /* Panel: strip all spacing, content-driven height */
  .hero-right-panel {
    width: 100%;
    flex: 0 0 auto;
    min-height: 0;          /* Was 54vh — caused massive gap below image */
    padding: 0;
    margin-top: 0.125rem;   /* Tiny gap between Section 1 and 2 */
    align-items: stretch;
    justify-content: stretch;
  }

  /* Wrapper: aspect-ratio locks to image proportions */
  .hero-slideshow-wrapper {
    padding: 0;
    border-radius: 0;
    box-shadow: none;
    border: none;
    background: #1a0a2e;
    aspect-ratio: 5 / 2;    /* Matches 2000x800 image → zero crop, zero gaps */
    width: 100%;
    height: auto;
  }

  /* Kill decorative top-edge highlight */
  .hero-slideshow-wrapper::before {
    display: none;
  }

  .hero-slideshow-wrapper img {
    border-radius: 0 !important;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
