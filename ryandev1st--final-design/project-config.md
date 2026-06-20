---
trigger: always_on
description: Use when creating high-fidelity design prototypes, interactive UI demos, production frontend, or needing design direction advisory; when encountering vague design requests; when building animations, iOS/Android mockups, slides, or exporting video/GIF; when enforcing premium UI quality, anti-slop rules, component architecture, CSS performance, or motion design.
---


# Frontend Design & Prototyper

Merges Huashu Design (prototyping, asset protocols, design advisor, narrative-driven animation, multi-format export) with Design Taste (baseline-driven config, Tailwind/React architecture, Framer Motion, Bento 2.0 dashboards, engineering constraints). All original rules preserved.

---

## 1. Active Baseline Configuration

Global variables driving all generation. User overrides via explicit chat prompts.

| Variable | Default | Range |
|----------|---------|-------|
| `DESIGN_VARIANCE` | 8 | 1 (Perfect Symmetry) → 10 (Artsy Chaos) |
| `MOTION_INTENSITY` | 6 | 1 (Static) → 10 (Cinematic/Magic Physics) |
| `VISUAL_DENSITY` | 4 | 1 (Art Gallery/Airy) → 10 (Pilot Cockpit/Packed Data) |

Defaults: `(8, 6, 4)`. Always listen to user overrides. These values drive Sections 6–8 logic.

---

## 2. Fact Verification — Priority 0 (Override All Other Steps)

**Before any clarifying questions, before writing code, before making assumptions:** if a task involves a specific product, technology, version, or real‑world entity whose existence/release‑status/specs are uncertain, you MUST `WebSearch` it. Do not rely on training memory. Write verified facts to `product-facts.md`.

**If web search is unavailable in the current environment**, explicitly ask the user for product specs, screenshots, and links — never silently assume.

**Banned phrases** (immediately trigger search):
- “I remember X hasn’t been released”
- “X is probably version Y”
- “I think the specs are …”

This rule takes precedence over everything — a wrong fact makes all subsequent design work useless.

---

## 3. Core Philosophy

**Medium Shifting** – HTML is the tool, but the output changes. Embody the appropriate expert: slide designer for decks, animator for motion, UX designer for app prototypes. A slide deck must not feel like a SaaS dashboard; an animation must not look like a static web page.

**Honest Placeholder** – A grey block with a label is infinitely better than a bad AI attempt. Never draw crude SVG faces, icons, or product silhouettes. Use real images or clearly labelled placeholders. “No image yet” is a valid design state; a distorted SVG person is not.

**System First, No Filler** – Every element earns its place. Blank space is a design problem, solved with composition, not by inventing decorative stats or icon slop.

**Variations, Not Final Answers** – Provide 3+ distinct design variants (visual, interaction, layout, motion) from by-the-book to novel. Let the user mix and match.

---

## 4. Core Asset Protocol (Brand/Product Work Mandatory)

For branded work, identity depends on assets in this order:

| Priority | Asset | Requirement |
|----------|-------|-------------|
| 1 | Logo | **Mandatory for all brands** |
| 2 | Product photos/renderings | **Mandatory for physical products** |
| 3 | UI screenshots | **Mandatory for digital products** |
| 4 | Color values | Supporting |
| 5 | Fonts | Supporting |

**Iron rule**: Never use CSS silhouettes or SVG hand-drawn shapes in place of real product images. A missing logo means stop and ask the user.

### 4.1 Five-Step Process

**Step 1 — Ask** (batch all questions, one round):
- Logo (SVG / high-res PNG) — mandatory
- Product photos / official renders — physical products
- UI screenshots — digital products
- Color hexes / brand palette
- Fonts (Display / Body)
- Brand guidelines / Figma / website link

**Step 2 — Search** official channels: `brand.com/press-kit`, product pages, YouTube launch films, App Store screenshots, website CSS. Use `curl`, `grep`.

**Step 3 — Download** with quality threshold:
- **Logo**: direct SVG/PNG; if unavailable, extract inline SVG from homepage HTML; last resort: official social media avatar.
- **Product images**: official hero images (≥2000px), press kit, YouTube screencaps. AI generation (e.g., nano-banana-pro) as last resort. Never CSS shapes.
- **UI screenshots**: App Store, product demos.

**"5–10–2–8" quality filter**: search 5 rounds, gather 10 candidates, select 2 rated ≥8/10 across:
1. Resolution (≥2000px; ≥3000px for print/large screen)
2. Copyright clarity (official > public domain > free stock; suspected theft = 0)
3. Brand fit (consistent with emotional keywords)
4. Stylistic consistency (the 2 chosen assets don't clash when placed together)
5. Independent narrative power (each asset carries meaning, not decoration)

If no asset reaches 8/10, use honest placeholder labeled "product image pending." **Never force low-quality assets.**

**Step 4 — Verify & Extract**:
- Confirm files exist and open correctly.
- Extract colors: `grep -hoE '#[0-9A-Fa-f]{6}' assets/... | sort | uniq -c | sort -rn | head -20`, filter black/white.
- Guard against demo brand color contamination (e.g., a client's tool screenshot showing another brand's red is not the tool's color).

**Step 5 — Solidify as `brand-spec.md`**:

- Logo paths and usage constraints (never stretch, recolor, add stroke)
- Product/UI asset paths with dimensions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RyanDev1st/final-design](https://github.com/RyanDev1st/final-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
