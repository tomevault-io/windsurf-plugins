---
trigger: always_on
description: > Operational contract for every agent and contributor building Onda. Read fully before generating anything. If a request conflicts with these rules, the rules win. When in doubt, choose restraint.
---

# CLAUDE.md — Onda

> Operational contract for every agent and contributor building Onda. Read fully before generating anything. If a request conflicts with these rules, the rules win. When in doubt, choose restraint.

**See also:** [README.md](README.md) · [docs/](docs/) · [docs/techspecs/](docs/techspecs/)

Onda is a Remotion-based motion graphics library: component source is copied into the user's project via our own thin CLI (`npx ondajs add <name>`), never imported as a black-box dependency. The differentiator is a **signature motion identity**: a consistent set of motion fingerprints, applied across ordinary components, that makes an Onda animation recognizable by feel. Full context: [docs/vision.md](docs/vision.md), [docs/design-philosophy.md](docs/design-philosophy.md), [docs/motion-language.md](docs/motion-language.md).

-----

## 1. Hard technical rules (non-negotiable — violating these breaks renders)

Remotion renders deterministically: the same frame must always produce the same pixels.

- **NEVER use `Math.random()`, `Date.now()`, `new Date()`,** or any non-deterministic value in render. Need randomness? Derive it from a seed prop via a pure function keyed off `frame`/index.
- **NEVER use `useState`/`useEffect` to drive animation.** All motion is a pure function of `useCurrentFrame()`. State/effects are for nothing visual.
- **Read time via `useCurrentFrame()` and config via `useVideoConfig()`** — never assume fps or dimensions.
- A component must render **frame N correctly with zero knowledge of frames 0…N-1.**
- **SSR-safe:** no `window`/`document` access during render without a guard.
- Wrap timed sections in `<Sequence>`; use `<AbsoluteFill>` for full-canvas layers.
- Audio/video go through `@remotion/media`, used correctly.

-----

## 2. Design tokens (locked — use these exact values)

**In-code canonical source:** [lib/tokens.ts](lib/tokens.ts). The values below mirror it; divergence is a bug.

These power BOTH the docs site and the default look of components. Expose them as CSS variables / a theme object; never hardcode raw values when a token exists.

### Color

```
--onda-bg          #08080A   /* near-black canvas — motion reads best on dark */
--onda-surface     #0E0E12   /* cards, raised surfaces */
--onda-surface-2   #121217   /* secondary surface */
--onda-border      #1C1C22
--onda-border-lit  #26262E   /* hover / focus borders */
--onda-text        #F2F2F4   /* primary text */
--onda-dim         #8E8E98   /* secondary text */
--onda-faint       #56565F   /* labels, captions, code prompts */
--onda-accent      #D96B82   /* THE accent — muted dusty rose. Used sparingly. */
--onda-accent-soft #E89AAB   /* lighter step, subtle depth only */
```

**Color rule:** the accent is used *sparingly* — one headline word, a number, an underline, a CTA, a single glow. Everything else is neutral. Color is earned, never sprinkled.

### Type

```
Display / headings:  "Clash Display"  (characterful geometric; weights 500–600)
Body / UI / mono:    "Space Grotesk"  (clean, technical; weights 400–600)
```

- Headlines: tight letter-spacing (-0.02 to -0.04em), weight 600.
- Labels / code / captions: Space Grotesk, uppercase for eyebrows, letter-spacing ~0.06–0.16em.
- Components accept `fontFamily` as a prop; default to the display font via `THEME.fontDisplay` (which falls back to Clash Display). **Never** hardcode Inter/Roboto/Arial/system fonts as a default (those read as generic AI defaults).

### Spacing

- 8px base scale (8/16/24/32/48/64/80/100).
- Scene-block safe margins: ~10% of canvas per edge. Negative space is part of the identity.

### Surface polish

- Cards: subtle top sheen (1px white-alpha gradient), soft deep shadow (`0 30px 60px -34px rgba(0,0,0,0.9)`), 1px border, ~20px radius.
- One restrained accent glow per major section max — depth, not decoration.
- Optional ~2% grain overlay for texture. Never busy.

### Theming — brand overrides (CSS variables)

The token values above are the **default** look, not an enforced one. Tokens are exposed as CSS custom properties so any consumer can re-skin components with their own brand. Full guide: [docs/theming.md](docs/theming.md).

- [lib/tokens.ts](lib/tokens.ts) defines `CSS_VAR` (the property names, e.g. `--onda-accent`) and `THEME` — `var()` strings with the Onda token as fallback (`THEME.accent === 'var(--onda-accent, #D96B82)'`).
- **Components default their color / font props to the `THEME` tokens, never a raw hex / font string.** So an unset brand renders identically to the values above, and setting the matching variable re-skins everything with zero per-component work. When authoring a component, use `THEME.*` for every color/font default (this is what §4.3 "premium defaults using the tokens" means).
- Apply a brand with `brandToCssVars(brand)` (spread onto a root element), `<ThemeProvider brand={…}>`, or `CompositionRenderer`'s `brand` prop. Consumers can also just set the `--onda-*` variables in their own CSS — including pointing `--onda-font-display` / `--onda-font-body` at any font they've loaded in their project.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [degueba/onda](https://github.com/degueba/onda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
