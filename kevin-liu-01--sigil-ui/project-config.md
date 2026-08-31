---
trigger: always_on
description: > Imported from Kevin's wiki and the Dedalus design language. These rules are non-negotiable when writing or reviewing Sigil code.
---

# Sigil Design System — Enforced Rules

> Imported from Kevin's wiki and the Dedalus design language. These rules are non-negotiable when writing or reviewing Sigil code.
>
> **Full references:** [style/design.md](../../style/design.md) (motion guide) | [style/ux-principles.md](../../style/ux-principles.md) (product UX) | [style/style.md](../../style/style.md) (engineering philosophy) | [skills/sigil-polish/](../../skills/sigil-polish/SKILL.md) (interface polish skill with typography, surfaces, animations, performance deep-dives)
>
> **Anti-slop enforcement:** Also read [taste-enforcement.mdc](./taste-enforcement.mdc) for banned AI visual patterns, variance dials, content quality rules, performance guardrails, and output completeness requirements. The [taste-skills-index.mdc](./taste-skills-index.mdc) catalogs the 12 user-level taste skills (taste-core, taste-soft, taste-minimalist, taste-brutalist, etc.) that auto-trigger based on task description.
>
> **Companion rule files (imported from wiki):**
> - [css-ui-enforcement.mdc](./css-ui-enforcement.mdc) — Tailwind + `cn()` first, `globals.css` only, scrollbars, hit areas, no `style`+`className` soup
> - [react-conventions.mdc](./react-conventions.mdc) — Hooks, linting (Ultracite/oxlint), architecture-policy ESLint rules, Vercel 69 rules, RSC safety
> - [typescript-conventions.mdc](./typescript-conventions.mdc) — Types, error handling (`Result<T,E>`, Zod), console labels, tsgo, oxfmt
> - [design-animation-rules.mdc](./design-animation-rules.mdc) — Animation frequency framework, easing flowchart, springs, clip-path, gestures, accessibility, Sonner principles
> - [frontier-stack.mdc](./frontier-stack.mdc) — Default animation/3D/rendering stack, decision matrix, json-render for generative UI
> - [dashboard-design.mdc](./dashboard-design.mdc) — Dashboard design system (14px base, stat cards, charts, sidebar, empty states, 6-rule cheatsheet)
> - [interface-micro-polish.mdc](./interface-micro-polish.mdc) — Dark 1px card shadow, image outlines, icon animation, staggered enters, softer exits

## Custom Preset Rule (MANDATORY — Enforced Before All Other Rules)

**Every custom preset MUST populate ALL 33 token categories and ALL ~519 fields.**

- The canonical template is `packages/presets/src/_template.ts`.
- When creating a custom preset: copy `_template.ts` or spread it as a base. Change values, never delete fields.
- No partial presets. If a field exists in `_template.ts`, it must exist in your preset.
- The 33 required categories: `colors`, `typography`, `spacing`, `layout`, `sigil`, `radius`, `shadows`, `motion`, `borders`, `buttons`, `cards`, `headings`, `navigation`, `backgrounds`, `code`, `inputs`, `cursor`, `scrollbar`, `alignment`, `sections`, `dividers`, `gridVisuals`, `focus`, `overlays`, `dataViz`, `media`, `controls`, `componentSurfaces`, `hero`, `cta`, `footer`, `banner`, `pageRhythm`.
- Read `skills/sigil-preset/SKILL.md` before creating any preset.

## Color Rules

- OKLCH for all authored palettes. Use `oklch(L C H)` — L=lightness (0-1), C=chroma (0-0.37), H=hue (0-360).
- At most 3 active colors plus neutrals per preset.
- Rich Black (oklch ~0.08) for dark mode page backgrounds, never pure #000000.
- Five text hierarchy levels: `--s-text`, `--s-text-secondary`, `--s-text-muted`, `--s-text-subtle`, `--s-text-disabled`. Always use the right level.
- Four border levels: `--s-border`, `--s-border-muted`, `--s-border-strong`, `--s-border-interactive`.
- BANNED: Hardcoded hex colors in components. All colors must reference `var(--s-*)` tokens.
- BANNED: Gradients with no material logic. Every gradient must have a reason (glow, light, depth, brand).

## Contrast Rules (WCAG AA)

All presets MUST pass WCAG 2.0 AA contrast requirements. Run `pnpm audit:contrast` before shipping any preset change.

| Pair | Minimum Ratio | WCAG Category |
|------|:---:|---|
| Normal text on background/surface | 4.5:1 | AA normal text |
| Large text (>= 18pt) on background | 3:1 | AA large text |
| UI components (borders, icons, form controls) on background | 3:1 | AA UI components |
| Button text (`primary-contrast`) on `primary` | 4.5:1 | AA normal text |
| Status colors (success/warning/error/info) on background | 3:1 | AA UI components |

Rules:
- `text-muted` on `background` must have >= 4.5:1 contrast ratio in BOTH light and dark modes.
- `text-subtle` is decorative/large text — >= 3:1 minimum.
- `border` and `border-strong` on `background` must have >= 3:1 for visibility.
- When `primary` is light (L > 0.55 in OKLCH), `primary-contrast` MUST be black (`oklch(0 0 0)` or `#000000`).
- When `primary` is dark (L <= 0.55), `primary-contrast` MUST be white (`oklch(1 0 0)` or `#ffffff`).
- Never assume white text on colored backgrounds — always check the contrast ratio.
- The audit script checks 30 color pairs per preset across both modes (930 checks total).

## Typography Rules

- Font triad: display face for headlines only, body face for everything else, mono for code/labels/data.
- Do not let all three compete equally in a single view.
- `text-wrap: balance` on all short headings and marketing copy.
- `-webkit-font-smoothing: antialiased` is set globally — never override to `auto`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kevin-Liu-01/Sigil-UI](https://github.com/Kevin-Liu-01/Sigil-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
