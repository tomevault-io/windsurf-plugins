---
trigger: always_on
description: Create stunning, branded social media designs as single-page HTML files — Instagram posts/stories/carousels, LinkedIn banners, Twitter cards, and more. Supports Unsplash API for automatic image sourcing. Helps non-designers discover their aesthetic through visual exploration rather than abstract choices.
---


# Frontend Social Media Design Skill

Create zero-dependency, visually striking HTML designs for social media content that can be screenshotted or exported as images. This skill helps non-designers discover their preferred aesthetic through visual exploration ("show, don't tell"), then generates production-quality social media graphics.

## Core Philosophy

1. **Zero Dependencies** — Single HTML files with inline CSS/JS. No npm, no build tools.
2. **Show, Don't Tell** — People don't know what they want until they see it. Generate visual previews, not abstract choices.
3. **Distinctive Design** — Avoid generic "AI slop" aesthetics. Every design should feel custom-crafted.
4. **Production Quality** — Code should be well-commented, accessible, and pixel-perfect at target dimensions.
5. **Canvas Sizing (CRITICAL)** — Every design MUST use exact fixed dimensions matching the target social media platform. No scrolling, no overflow.

---

## CRITICAL: Canvas Sizing Requirements

**This section is mandatory for ALL designs. Every canvas must be pixel-perfect at the target platform dimensions.**

### The Golden Rule

```
Each design = exactly one fixed-size canvas (e.g., 1080×1080px)
Content overflows? → Split into carousel pages or simplify
Never scroll within a canvas.
```

### Platform Dimensions

| Platform | Format | Dimensions (px) |
|----------|--------|-----------------|
| Instagram | Post (Square) | 1080 × 1080 |
| Instagram | Post (Portrait) | 1080 × 1350 |
| Instagram | Story / Reel | 1080 × 1920 |
| Instagram | Carousel Page | 1080 × 1080 or 1080 × 1350 |
| Facebook | Post | 1200 × 630 |
| Twitter / X | Post | 1200 × 675 |
| LinkedIn | Post | 1200 × 628 |
| LinkedIn | Carousel Page | 1080 × 1080 or 1080 × 1350 |
| YouTube | Thumbnail | 1280 × 720 |
| Pinterest | Pin | 1000 × 1500 |
| TikTok | Cover | 1080 × 1920 |

### Content Density Limits

To guarantee content fits, enforce these limits per canvas/page:

| Content Type | Maximum Content |
|--------------|----------------|
| Quote post | 1 quote (max 3 lines) + attribution + optional logo |
| Headline post | 1 headline + 1 subtitle + CTA |
| Feature post | 1 headline + 3-4 bullet points or icons |
| Image post | 1 headline + 1 image (max 60% canvas height) + optional caption |
| Carousel page | 1 heading + 3-4 points OR 1 heading + 1 image + caption |
| Infographic | 1 heading + 4-6 data points/icons |

**If content exceeds these limits → Split into carousel pages**

### Required CSS Architecture

Every design MUST include the base CSS from `STYLE_PRESETS.md`. Key requirements:

- `.canvas` has exact `width` and `height` in pixels
- `overflow: hidden` prevents any content from escaping
- `container-type: inline-size` enables container queries for responsive typography
- All typography uses `clamp()` with container-query-relative units (`cqw`)
- Carousel designs use `.carousel` + `.carousel-track` + `.carousel-page` structure

**See STYLE_PRESETS.md for complete base CSS, carousel CSS, and Unsplash overlay CSS.**

### When Content Doesn't Fit

**DO:**
- Split into carousel pages
- Reduce text (shorter headlines, fewer bullets)
- Use icons instead of text where possible
- Create a "continued" page in the carousel

**DON'T:**
- Reduce font size below readable limits
- Remove padding/spacing
- Allow any overflow
- Cram content to fit

---

## Phase 0: Detect Mode

First, determine what the user wants:

**Mode A: Single Post**
- User wants to create a single social media graphic
- Proceed to Phase 1 (Content Discovery)

**Mode B: Carousel**
- User wants a multi-page carousel (Instagram, LinkedIn, etc.)
- Proceed to Phase 1, with carousel-specific questions

**Mode C: Batch Generation**
- User wants multiple designs (e.g., a set of quote posts, a content series)
- Proceed to Phase 1, clarify how many and what varies between them

**Mode D: Existing Design Enhancement**
- User has an HTML design and wants to improve it
- Read the existing file, understand the structure, then enhance

### Mode D: Critical Modification Rules

When enhancing existing designs, follow these mandatory rules:

**1. Before Adding Any Content:**
- Check current content against density limits
- Calculate if new content will fit within the canvas

**2. When Adding Images:**
- Images must have `max-height: 60%` or similar canvas constraint
- If current canvas is already full → split into carousel
- Unsplash images need an overlay for text readability

**3. Required Checks After ANY Modification:**
```
✅ Does the canvas have `overflow: hidden`?
✅ Are all new elements using appropriate font sizes?
✅ Do new images have canvas-relative max-height?
✅ Does total content respect density limits?
✅ Is Unsplash attribution included if using API photos?
```

---

## Phase 1: Content Discovery

Before designing, understand the content.

### Step 1.1: Design Context (Single Form)

**IMPORTANT:** Ask ALL questions in a single AskUserQuestion call.

**Question 1: Platform**
- Header: "Platform"
- Question: "Which platform is this for?"
- Options:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dikaizm/frontend-social-media-design](https://github.com/dikaizm/frontend-social-media-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
