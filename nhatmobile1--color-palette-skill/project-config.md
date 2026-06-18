---
trigger: always_on
description: Create distinctive, accessible color palettes for UI/web design that avoid generic AI aesthetics. Use when designing websites, applications, or any digital interface requiring thoughtful color selection. Provides curated domain-specific palettes, color theory guidance, accessibility validation, and strategies to break away from overused patterns (purple gradients, orange-teal combinations, generic tech blues). Includes contrast checkers, palette generators, and comprehensive reference materials 
---


# Color Palette Creation

## Overview

This skill helps create distinctive, accessible color palettes for UI and web design that stand out from generic AI-generated aesthetics. It provides color theory guidance, curated domain-specific examples, accessibility validation tools, and strategies to avoid overused patterns.

## When to Use This Skill

Use this skill when:
- Designing color palettes for websites, apps, or digital interfaces
- Users request color schemes for specific domains or industries
- Building design systems or style guides
- Ensuring accessibility compliance (WCAG AA/AAA)
- Breaking away from generic "AI-looking" designs
- Need validation of existing color choices

## Workflow

### Step 1: Understand Context and Requirements

Before selecting or generating colors, gather essential context:

**Domain/Industry Questions:**
- What industry or domain is this for? (Tech/SaaS, E-commerce, Healthcare, Finance, Creative, Food, etc.)
- What emotions or associations should the palette convey?
- Are there existing brand colors to work with or extend?

**Technical Requirements:**
- Light mode, dark mode, or both?
- Accessibility requirements? (WCAG AA minimum, AAA preferred)
- How many colors needed? (Typically: 1-2 primary, 1-2 secondary, 1-2 accents, full neutral scale, semantic colors)

**Distinctiveness Goals:**
- Should this avoid looking "AI-generated"?
- Any specific colors or combinations to avoid?
- Preference for warm vs cool tones?

### Step 2: Select Base Approach

Choose one of these approaches based on the context:

#### Approach A: Domain-Specific Curated Palettes

**When to use:** Clear industry/domain, want proven effective combinations

**Process:**
1. Read `references/color-palette-ui-design-reference.md` focusing on the relevant domain section
2. Present 2-3 palette options from that domain
3. Explain what makes each effective
4. Adapt the chosen palette if needed

**Example domains:**
- Tech/SaaS: Trust & stability, Modern minimal, Professional dashboards, Dark mode
- E-commerce: Fashion, Beauty, Sports, Home, Luxury
- Healthcare: Calm professional, Soothing apps, Mental health/wellness
- Finance: Traditional banking, Modern fintech, Investment platforms
- Creative/Portfolio: Bold & modern, Minimalist, Artistic, Developer portfolios
- Food/Restaurant: Warm & appetizing, Delivery apps, Organic, Upscale

#### Approach B: Generate from Brand Color

**When to use:** Starting from existing brand color(s), need to build complete system

**Process:**
1. Use `scripts/generate_palette.py` to create color scales:
   ```bash
   python scripts/generate_palette.py #3B82F6 scale
   ```
2. Generate harmonious colors using color theory:
   ```bash
   python scripts/generate_palette.py #3B82F6 complementary
   python scripts/generate_palette.py #3B82F6 triadic
   python scripts/generate_palette.py #3B82F6 analogous
   ```
3. Validate contrast ratios (Step 3)
4. Apply 60-30-10 rule (see reference document)

#### Approach C: Custom Palette from Inspiration

**When to use:** Need unique, distinctive palette; have specific inspiration sources

**Process:**
1. If user provides inspiration (image, artwork, location), extract colors from that source
2. Review "Common AI Design Pitfalls" section in references to avoid generic patterns
3. Apply color harmony principles from reference document
4. Use "Anti-AI Checklist" to ensure distinctiveness
5. Validate accessibility (Step 3)

### Step 3: Validate Accessibility

**Always validate contrast ratios** using the contrast checker script:

```bash
python scripts/check_contrast.py <foreground> <background> <text-size>
```

**Examples:**
```bash
# Check normal text on background
python scripts/check_contrast.py #1A1A1A #FFFFFF normal

# Check large text (headings)
python scripts/check_contrast.py #4A4A4A #FFFFFF large

# Check button contrast
python scripts/check_contrast.py #FFFFFF #3B82F6 normal
```

**Requirements:**
- Normal text: 4.5:1 minimum (AA), 7:1 preferred (AAA)
- Large text (18pt+): 3:1 minimum (AA), 4.5:1 preferred (AAA)
- UI components (buttons, borders, icons): 3:1 minimum

**If validation fails:**
- Darken the foreground color
- Lighten the background color
- Use the palette generator to create darker/lighter shades
- Consider alternative color combinations

### Step 4: Structure Complete Color System

Organize the palette following design system best practices:

#### Primary Colors
- Main brand identity (1-2 colors)
- Full scale (50-950 shades)
- Use for: Logo, primary buttons, active states

#### Secondary Colors
- Supporting brand colors (1-3 colors)
- Full or partial scale depending on usage
- Use for: Secondary buttons, icons, less prominent features

#### Accent Colors
- Attention-drawing colors (1-2 colors)
- Apply 60-30-10 rule: Use for only 10% of the design

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nhatmobile1/color-palette-skill](https://github.com/nhatmobile1/color-palette-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
