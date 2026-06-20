---
trigger: always_on
description: >
---


# ANSI Art Creation Guide

This skill teaches you how to create compelling ANSI art using the `window.ansi` programmatic API. It covers the artistic fundamentals (color, shadow, composition) and the technical workflow (API patterns, layering, reference conversion).

Read `community-techniques.md` for the full catalog of advanced drawing techniques from community tutorials. Read `material-optics-reference.md` for material optical properties used in Phase 6 mask generation.

## The CGA Palette: Your Paint Box

You have exactly 16 colors. Learning their relationships is the single most important thing for making good ANSI art.

### Color Groups by Luminance

Think of the palette in 4 brightness tiers:

| Tier | Colors | Indexes | Use |
|------|--------|---------|-----|
| **Black** | Black | 0 | Deepest shadow, outlines, negative space |
| **Dark** | Blue, Green, Cyan, Red, Magenta, Brown | 1-6 | Shadows, base tones, backgrounds |
| **Medium** | Dark Gray, Light Blue, Light Green, Light Cyan, Light Red, Light Magenta, Yellow | 7-14 | Midtones, highlights on dark areas |
| **Bright** | White | 15 | Brightest highlights, specular, text |

Dark Gray (8) is your most versatile shadow color — it works with almost everything.

### Natural Color Pairs (shadow → highlight)

These are the pairs that create convincing shading when placed adjacent:

- **Blues:** 1 (dark blue) → 9 (light blue) → 15 (white)
- **Greens:** 2 (green) → 10 (light green) → 15
- **Cyans:** 3 (cyan) → 11 (light cyan) → 15
- **Reds:** 4 (red) → 12 (light red/pink) → 15
- **Magentas:** 5 (magenta) → 13 (light magenta) → 15
- **Browns/Yellows:** 6 (brown) → 14 (yellow) → 15
- **Grays:** 0 (black) → 8 (dark gray) → 7 (light gray) → 15 (white)

**The gray ramp** (0 → 8 → 7 → 15) is essential. It provides 4 neutral tones for any subject.

### Color Relationships for Skin/Flesh Tones

ANSI art has no peach or beige. Artists approximate skin using:
- **Light skin:** Brown (6) base, Yellow (14) highlight, Dark gray (8) shadow
- **Dark skin:** Red (4) or Brown (6) base, Light red (12) midtone, Dark gray (8) shadow
- **Blue/fantasy skin:** Cyan (3) base, Light cyan (11) highlight, Blue (1) shadow
- **Metallic/armor:** Dark gray (8) base, Light gray (7) midtone, White (15) specular

### The Brown Problem

Brown (6) and Yellow (14) are the same hue at different brightnesses. Brown renders as dark orange/brown on screen. There's no background yellow — only background brown. This means yellow text on brown background becomes invisible. Plan around this.

## Shadow & Light Theory

### Establishing a Light Source

Before drawing anything, decide where light comes from. Upper-left is the most natural default. Then apply consistently:

- **Lit faces:** Use the bright variant of your base color
- **Shadow faces:** Use the dark variant — but match the hue (see below)
- **Ambient occlusion:** Where surfaces meet (under chin, inside ears, behind hair), go to black (0)
- **Specular highlights:** Tiny white (15) spots where light hits directly

### Color-Appropriate Shadows

This is critical and easy to get wrong: shadow colors must match the hue of the surface they're on. Dark gray (8) is versatile but it's a neutral — using it on a strongly colored surface kills the color identity.

- **Blue skin/surfaces:** Use blue (1) for shadows, not dark gray (8). Gray shadows make blue skin look lifeless and gray.
- **Green surfaces:** Use green (2) for shadows. Dark gray strips the natural feel.
- **Red surfaces:** Use red (4) for shadows. Gray shadows look like the red is fading.
- **Warm skin (brown/yellow):** Dark gray (8) works here because brown is already desaturated.
- **Metallic/neutral surfaces:** Dark gray (8) is correct — metal is inherently neutral.

The rule: if the subject has a strong hue identity, shadow with the dark variant of that hue, not with neutral gray. Only use gray shadows on surfaces that are already neutral.

### The Three-Tone Rule

Every surface needs at minimum 3 tones to read as 3D:
1. **Shadow tone** — the darkest value on that surface
2. **Base tone** — the dominant mid-value
3. **Highlight tone** — where light strikes

Example for a blue sphere: Blue (1) shadow → Cyan (3) base → Light Cyan (11) highlight, with a single White (15) specular dot.

### Shadow Characters

Shade blocks create gradual transitions between tones:

| Character | Code | Name | Density |
|-----------|------|------|---------|
| `░` | 176 | Light shade | ~25% filled |
| `▒` | 177 | Medium shade | ~50% filled |
| `▓` | 178 | Dark shade | ~75% filled |
| `█` | 219 | Full block | 100% filled |

Use them in sequence for smooth gradients. A transition from light to dark might go:
`░` (light fg on dark bg) → `▒` → `▓` → `█` (solid dark)

The foreground/background color combo on shade characters creates a blended appearance. For example, light cyan foreground + blue background on `▒` creates a mid-tone blue-cyan that neither solid color provides.

#### Shade Blocks for Glow Effects

Shade blocks excel at creating atmospheric glow around bright/energy sources. The technique:
1. Identify the bright energy source (a light, fire, magic, or other luminous element)
2. Apply BFS distance mapping outward from bright cells to nearby dark cells

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sigma-zenko/ansi-skill](https://github.com/sigma-zenko/ansi-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
