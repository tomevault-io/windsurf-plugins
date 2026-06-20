---
trigger: always_on
description: Create **bold, distinctive, production-grade** frontend interfaces. This skill produces expressive, modern designs that look nothing like generic AI output.
---

# Bold Designs Skill

Create **bold, distinctive, production-grade** frontend interfaces. This skill produces expressive, modern designs that look nothing like generic AI output.

---

## When to Apply This Skill

Use this when:
- Building landing pages, marketing sites, or product interfaces
- Creating UI components (buttons, cards, heroes, navigation)
- Writing frontend code in React, Next.js, Vue, Svelte, or plain HTML/CSS
- Generating Tailwind CSS or vanilla CSS

---

## ⚠️ Project Styling Respect Rule

**This skill provides PRINCIPLES and TACTICS, not specific styling.**

When working in an existing project:
1. **ALWAYS** use the project's existing design system, colors, and tokens
2. **ALWAYS** match the project's existing spacing scale and typography
3. **ALWAYS** follow the project's component patterns and naming conventions
4. **NEVER** override existing Tailwind config, CSS variables, or theme settings
5. **NEVER** introduce new colors/fonts that conflict with the established palette

**What this skill DOES provide:**
- Anti-generic aesthetic principles (what to avoid, what makes designs feel templated)
- Accessibility requirements (APCA contrast, keyboard nav, ARIA)
- Animation performance rules (compositor properties, timing)
- Typography best practices (`text-balance`, `text-pretty`, hierarchy)
- Layout discipline (z-index scale, safe areas, optical alignment)
- Interaction patterns (focus-visible, touch targets, states)

**What this skill does NOT impose:**
- Specific color palettes (use the project's palette)
- Specific fonts (use the project's typography)
- Specific component library (adapt to project's stack)
- Specific spacing values (use the project's scale)

The color palettes and component examples below are **reference examples only** — use them for new/greenfield projects, or when no existing design system exists.

---

## Design Philosophy

### Core Principles

1. **Bold over safe** — Make visual choices that have a point of view. Boring is worse than slightly wrong.
2. **Intentional over uniform** — Every spacing, color, and size decision should serve a purpose.
3. **Expressive over minimal** — Users want personality, not sterile tech aesthetics.
4. **Readable over clever** — Visual interest must never sacrifice readability.
5. **Native over custom** — Use semantic HTML and CSS before JavaScript solutions.

### Anti-Generic Rules

**NEVER do these (they scream "AI generated"):**
- Uniform padding/margins everywhere (8px everywhere = AI)
- Default blue buttons without context
- Overly rounded corners on everything (rounded-2xl on everything = AI)
- Generic gradients (blue-to-purple is the new clipart)
- Centered everything with identical spacing
- Stock "hero + 3 features + testimonials + CTA" layout
- Sans-serif body text with no typographic personality
- Identical card components repeated without variation

**ALWAYS do these:**
- Mix spacing intentionally (tight headlines, generous section breaks)
- Use asymmetry where it serves hierarchy
- Add one unexpected visual element per section
- Vary component sizes based on importance
- Use color strategically, not decoratively
- Create visual rhythm through contrast, not repetition

---

## Visual Design System

### Typography

**Hierarchy (most important → least):**
| Level | Use | Size | Weight | Line Height |
|-------|-----|------|--------|-------------|
| Display | Hero headlines | 4xl-7xl | 700-900 | 1.0-1.1 |
| H1 | Page titles | 3xl-5xl | 700-800 | 1.1-1.2 |
| H2 | Section heads | 2xl-3xl | 600-700 | 1.2 |
| H3 | Card titles | xl-2xl | 600 | 1.3 |
| Body Large | Intro paragraphs | lg-xl | 400 | 1.6-1.7 |
| Body | Default text | base | 400 | 1.6 |
| Small | Captions, labels | sm | 500 | 1.4 |

**Typography Rules:**
- Apply `text-balance` to all headings
- Apply `text-pretty` to body paragraphs
- Use `tabular-nums` for any numerical data
- Use `truncate` or `line-clamp` for dense UI
- Never modify letter-spacing unless explicitly requested
- Font size ≥16px on mobile to prevent iOS auto-zoom
- Curly quotes (" ") not straight quotes (" ")
- Ellipsis character (…) not three periods (...)

**Font Pairing Strategy:**
- Headlines: Bold geometric sans (Inter, Satoshi, Plus Jakarta) or expressive display (Clash Display, Cabinet Grotesk)
- Body: Readable neutral sans (Inter, System UI) or humanist (Source Sans, Open Sans)
- Accent: Monospace for technical content, serif for editorial

### Color Strategy

**Palette Approach:**
- Bold primaries as accents, not backgrounds
- High contrast text (APCA preferred over WCAG 2)
- One accent color per view maximum
- Neon/vibrant colors for CTAs and highlights
- Dark mode should feel native, not inverted

**Contrast Requirements:**
- Text on backgrounds: APCA Lc 75+ for body, Lc 60+ for large text
- Interactive states (hover/active/focus) must have HIGHER contrast than rest state
- Never rely on color alone — always include text labels or icons

**Color Palette Examples:**

```css
/* Electric & Bold */
--primary: #7C3AED;      /* Vibrant purple */
--accent: #06B6D4;       /* Cyan pop */
--surface: #0F172A;      /* Deep slate */
--text: #F8FAFC;         /* Bright white */

/* Warm Energy */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nielskaspers/bold-designs](https://github.com/nielskaspers/bold-designs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
