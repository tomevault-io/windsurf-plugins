---
trigger: always_on
description: Eliminates AI slop from frontend code. Combines design taste enforcement, anti-pattern detection, and structured remediation. Use this skill whenever generating or reviewing HTML, CSS, React, Vue, Svelte, or any frontend UI code.
---


# UIdetox — Combined Design Skill

## 1. ACTIVE BASELINE CONFIGURATION

* DESIGN_VARIANCE: 8 (1=Perfect Symmetry, 10=Artsy Chaos)
* MOTION_INTENSITY: 6 (1=Static/No movement, 10=Cinematic/Magic Physics)
* VISUAL_DENSITY: 4 (1=Art Gallery/Airy, 10=Pilot Cockpit/Packed Data)

**AI Instruction:** The standard baseline is strictly set to these values (8, 6, 4). Do not ask the user to edit this file. Adapt these values dynamically based on what the user explicitly requests. Use these values to drive the logic in Sections 3 through 8.

---

## 2. DESIGN DIRECTION

Commit to a BOLD aesthetic direction before writing a single line of code:
- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick an extreme — brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, industrial/utilitarian. There are infinite flavors.
- **Constraints**: Technical requirements (framework, performance, accessibility).
- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work — the key is intentionality, not intensity.

Then implement working code that is:
- Production-grade and functional
- Visually striking and memorable
- Cohesive with a clear aesthetic point-of-view
- Meticulously refined in every detail

---

## 3. DESIGN ENGINEERING DIRECTIVES (Bias Correction)

LLMs have statistical biases toward specific UI cliché patterns. Override them with these rules:

### Rule 1: Deterministic Typography

→ *Consult [typography reference](reference/typography.md) for scales, pairing, and loading strategies.*

* **Display/Headlines:** Default to large, tight tracking, reduced line-height. Headlines should feel heavy and intentional.
  * **ANTI-SLOP:** `Inter`, `Roboto`, `Arial`, `Open Sans`, system defaults are BANNED for creative or premium vibes. Force unique character using `Geist`, `Outfit`, `Cabinet Grotesk`, `Satoshi`, or a distinctive display font.
  * **TECHNICAL UI RULE:** Serif fonts are strictly BANNED for Dashboard/Software UIs. Use exclusively high-end Sans-Serif pairings (`Geist` + `Geist Mono` or `Satoshi` + `JetBrains Mono`).
* **Body/Paragraphs:** Readable sizes (14-16px body), limit paragraph width to ~65 characters, generous line-height.
* **Weight Spectrum:** Use Medium (500) and SemiBold (600), not just Regular and Bold.
* **Numbers:** Use monospace or enable `font-variant-numeric: tabular-nums` for data interfaces.
* **Letter-spacing:** Negative tracking for large headers, positive tracking for small caps or labels.
* **Don't** put large icons with rounded corners above every heading — they rarely add value and make sites look templated.

### Rule 2: Color Calibration

→ *Consult [color reference](reference/color-and-contrast.md) for OKLCH, palettes, and dark mode.*

* **Constraint:** Max 1 Accent Color. Saturation < 80%.
* **THE AI PALETTE BAN:** Purple/blue gradients, cyan-on-dark, neon accents on dark backgrounds — all BANNED. These are the fingerprints of AI-generated work. Use absolute neutral bases (Zinc/Slate) with high-contrast, singular accents (Emerald, Electric Blue, or Deep Rose).
* **COLOR CONSISTENCY:** Stick to one palette for the entire output. Never mix warm and cool grays. Tint all neutrals toward your brand hue.
* **GRAY ON COLOR:** Never put gray text on colored backgrounds — it looks washed out. Use a shade of the background color instead.
* **NO PURE BLACK:** Never use `#000000`. Use off-black, zinc-950, or tinted dark.
* **NO GRADIENT TEXT:** Do not use text-fill gradients for "impact" — especially on metrics or headings.
* Use modern CSS color functions (oklch, color-mix, light-dark) for perceptually uniform palettes.
* **Color Priority Order:**
  1. Use existing colors from the user's project (search for them by reading config files)
  2. Get inspired from the curated palettes in [color-palettes reference](reference/color-palettes.md)
  3. Never invent random color combinations

### Rule 3: Layout Diversification

→ *Consult [spatial reference](reference/spatial-design.md) for grids, rhythm, and container queries.*

* **ANTI-CENTER BIAS:** Centered Hero/H1 sections are BANNED when DESIGN_VARIANCE > 4. Force "Split Screen" (50/50), "Left Aligned content/Right Aligned asset", or "Asymmetric White-space" structures.
* **3-COLUMN CARD BAN:** The generic "3 equal cards horizontally" feature row is BANNED. Use a 2-column zig-zag, asymmetric grid, horizontal scroll, or masonry layout.
* **Container Constraint:** Always use max-width (1200-1440px) with auto margins.
* **Grid over Flex-Math:** Never use complex flexbox percentage math. Always use CSS Grid for reliable structures.
* **Viewport Stability:** Never use `h-screen` for full-height sections. Always use `min-h-[100dvh]`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OJamals/UIdetox](https://github.com/OJamals/UIdetox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
