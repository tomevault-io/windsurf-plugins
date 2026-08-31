---
trigger: always_on
description: > Adapted from [tasteskill.dev](https://www.tasteskill.dev/) for Sigil's token-driven architecture.
---

# Taste Enforcement — Anti-Slop Frontend Rules

> Adapted from [tasteskill.dev](https://www.tasteskill.dev/) for Sigil's token-driven architecture.
> These rules override default LLM biases that produce generic, forgettable frontends.
> Cross-reference with `sigil-design-system.mdc` for token consumption rules.
>
> **Available taste skills** (user-level at `~/.cursor/skills/taste-*/SKILL.md`):
> - `taste-core` — general anti-slop default
> - `taste-gpt` — GPT/Codex stricter variant
> - `taste-image-to-code` — image-first then implement
> - `taste-redesign` — audit existing UI
> - `taste-soft` — premium calm/expensive
> - `taste-output` — output completeness
> - `taste-minimalist` — editorial monochrome
> - `taste-brutalist` — Swiss/CRT/terminal
> - `taste-stitch` — Google Stitch DESIGN.md
> - `taste-imagegen-web` — web reference images
> - `taste-imagegen-mobile` — mobile screen images
> - `taste-brandkit` — brand-kit overview images
>
> See [taste-skills-index.mdc](./taste-skills-index.mdc) for the full selection guide.

## Variance Baseline

These dials drive all downstream decisions. Adjust per-prompt when the user specifies a different mood.

| Dial | Default | Range | What It Controls |
|------|:-------:|-------|-----------------|
| DESIGN_VARIANCE | 8 | 1-10 | Layout asymmetry, grid complexity, whitespace distribution |
| MOTION_INTENSITY | 6 | 1-10 | Animation density, spring physics, scroll effects |
| VISUAL_DENSITY | 4 | 1-10 | Spacing tightness, card usage, data presentation |

Interpret user requests dynamically: "make it airy" → VISUAL_DENSITY 2; "dashboard" → VISUAL_DENSITY 7-8; "cinematic" → MOTION_INTENSITY 8-9.

## Banned Visual Patterns (Hard Failures)

These patterns are the hallmark of generic AI output. Never produce them.

### Layout
- **Centered hero + blur blobs** when DESIGN_VARIANCE > 4. Use split-screen, left-aligned, or asymmetric whitespace.
- **3 equal cards in a row** for feature sections. Use 2-column zig-zag, asymmetric bento, or horizontal scroll.
- **`h-screen`** for full-height sections. Always use `min-h-[100dvh]`.
- **Flexbox percentage math** (`w-[calc(33%-1rem)]`). Use CSS Grid.
- **Complex layouts without mobile fallback.** Levels 4-10 variance MUST collapse to single-column below `md:`.

### Color & Surface
- **"AI Purple/Blue" aesthetic.** No purple button glows, no neon gradients. Use the active preset's `--s-primary`.
- **Pure `#000000`** for backgrounds or text. Sigil uses rich black via `var(--s-background)`.
- **Oversaturated accents.** Desaturate to blend with the preset's neutral scale.
- **Gradients with no material logic.** Every gradient needs a reason: glow, light source, depth, brand.
- **Glassmorphism on white** without functional justification.
- **Neon/outer glows** via default `box-shadow`. Use inner borders or tinted `var(--s-shadow-*)` tokens.
- **Excessive gradient text** on large headers.

### Typography
- **Inter, Roboto, Open Sans** as primary typeface for a new visual language. Sigil uses the PP Pangram collection or preset-specific stacks.
- **Oversized H1s** that scream instead of communicating hierarchy. Control with weight and color via `var(--s-heading-*)` tokens.
- **Serif fonts on dashboards/software UI.** Serif is for editorial/creative presets only.
- **Too many font families.** Max: the triad (display + body + mono).

### Content (The "Jane Doe" Effect)
- **Generic names:** "John Doe", "Jane Smith", "Sarah Chan", "Acme Corp" → invent realistic, contextual names.
- **Generic avatars:** No SVG egg icons or Lucide user placeholders → use styled initials or specific photo placeholders.
- **Fake round numbers:** `99.99%`, `50%`, `1234567` → use organic data (`47.2%`, `+1 (312) 847-1928`).
- **Startup slop names:** "Nexus", "SmartFlow", "SynergyAI" → invent premium, contextual brand names.
- **AI copywriting clichés:** "Elevate", "Seamless", "Unleash", "Next-Gen", "Game-changer", "Delve" → concrete verbs only.
- **Lorem ipsum** in any visible UI.
- **Emojis** in code, markup, headings, or alt text. Use Phosphor or Radix icons.

### External Resources
- **Unsplash links.** Use `https://picsum.photos/seed/{context}/W/H` or SVG placeholders.
- **Generic Lucide/Feather/Heroicons.** Use `@phosphor-icons/react` (Bold/Fill weights) or `@radix-ui/react-icons`.
- **Default shadcn/ui** without customization. All shadcn components MUST be restyled to match the active Sigil preset.

## Required UI States

LLMs default to the happy path. Every interactive component MUST include:

| State | Implementation |
|-------|---------------|
| Loading | Skeletal loaders matching layout dimensions — no generic spinners |
| Empty | Composed empty state explaining how to populate data |
| Error | Clear inline error reporting with `var(--s-error)` color |
| Active/Pressed | `-translate-y-[1px]` or `scale-[0.98]` for tactile feedback |
| Hover | Meaningful state change — not just opacity shift |

## Creative Proactivity

When MOTION_INTENSITY > 5, actively implement:

- **Spring physics** for all interactive elements: `type: "spring", stiffness: 100, damping: 20`. No linear easing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kevin-Liu-01/Sigil-UI](https://github.com/Kevin-Liu-01/Sigil-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
