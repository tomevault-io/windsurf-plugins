---
trigger: always_on
description: A design-method engine that makes AI reason like a strong UI/UX designer. StyleSeed fixes the
---

# StyleSeed — Design Engine

A design-method engine that makes AI reason like a strong UI/UX designer. StyleSeed fixes the
judgment process, not one aesthetic: a consumer-finance home, operations console, editorial
story, and commerce detail page use different output grammars. Skins provide materials; they
never define the method.

**Read `PRODUCT-PRINCIPLES.md` first.** Resolve every screen as core judgment × one output
grammar (`RULESETS.md` or a project-local grammar compiled with `/ss-reference`) × domain ×
page type × optional aesthetic profile (`PRESETS.md`) × bounded `STYLESEED.md` values.

## Golden Rules (NEVER break these)

```
 1. Use the selected output grammar's grouping model — cards, whitespace, rules, or tonal
    surfaces are tools, not a universal answer
 2. Color has stable roles and one identifiable primary action; extra hues require semantic,
    categorical, or brand meaning defined by the grammar
 3. No accidental pure black (#000); structural hard black is allowed only when an exact
    maintained grammar/profile contract calls for it
 4. Numbers 2:1 with units — 48px number + 24px unit, always
 5. One spatial rhythm on the 8px grid — mobile: space-y-6 · mx-6 · px-6; desktop: same
    principle via container + gap-6/gap-8 (don't mix off-grid one-offs)
 6. Never repeat same section type consecutively — create visual rhythm
 7. Elevation, one language: LIGHT = layered shadows ≤ 8% opacity (if visible, too strong);
    DARK = shadows don't read — tonal surface ramp (page < card < raised) + hairline borders
 8. Touch targets ≥ 44×44px on touch surfaces; pointer-first desktop controls may be 36–40px
    (keep visible focus rings either way)
 9. Semantic tokens only (text-brand, bg-card) — NEVER hardcode hex in components
10. Font sizes from the "Font Size by Context" table ONLY — don't guess
11. NO emoji as UI icons (🚗🧺⭐) — one line-icon set in currentColor; emoji inject many colors
12. Status color = severity only — a normal/"보통" state is grey, not colored; don't color every row
13. After generating ANY UI → run the Quality Gate (below); never show UI that hasn't passed
14. NEVER ship the default/unlocked accent (generic indigo #5E6AD2/#4F46E5) or a copied demo layout — lock a domain-fit key color + font FIRST (Quick Setup). A coherent-but-generic screen STILL reads "an AI made this"; coherent ≠ distinctive
15. One focal point per screen — the hero/primary element must visually dominate. An all-even grid of same-weight cards, centered and evenly spaced, is the #1 "machine-composed" tell
16. Match the type scale to the surface — mobile app uses the tight scale; desktop/web B2B uses the LARGER scale (body ≥16px). Don't ship 14px body on a 1440px screen
```

Reference this guide when Claude Code sets up a new project or implements UI.

> **When to read which file:**
> - **PRODUCT-PRINCIPLES.md**: Product constitution, authority order, fixed method vs variable
>   look. Read first.
> - **RULESETS.md**: Functional output grammars selected by the result's job. Read before domain
>   and page rules. Toss is one reference family, not the default for every result.
> - **ADAPTERS.md**: Surface/renderer contracts for product UI, carousels, decks, documents,
>   reports, and single-frame graphics.
> - **REFERENCE-COMPILER.md**: How `/ss-reference` turns user-supplied visual references into a
>   project-local evidence-backed grammar.
> - **PRESETS.md**: Optional aesthetic profiles for `/ss-restyle`; never a substitute for the
>   output grammar.
> - **ARCHITECTURE.md**: Engine flow, authority layers, grammar sources, and verification model.
> - **This file (CLAUDE.md)**: Tokens, component API, imports, forbidden patterns — reference while coding
> - **DESIGN-LANGUAGE.md**: Visual design rules, page layout, composition recipes — read **before** building a new page. Start with the Table of Contents, then rules 14, 18, 19, 61-63.
> - **METHODOLOGY.md**: UI/UX reasoning patterns (progressive disclosure, info density, atomic design, skeleton/empty/microinteraction, contextual onboarding, Linear/Toss aesthetic, color discipline, motion vibe vocabulary) — read **before scaffolding a new dashboard** or when wondering *why* the rules in DESIGN-LANGUAGE.md exist. Chapter 8 (Motion Vibe Vocabulary) is the entry point for the `engine/motion/` seed system.
> - **APP-PLAYBOOKS.md**: How to **bias** the rules for the app's domain (fintech, SaaS, e-commerce, social, content, productivity, health, education, dev-tools, marketplace, booking, AI/chat). Read **right after you know what kind of app this is** (e.g. from `/ss-setup`), before scaffolding — a fintech dashboard and a social dashboard apply the same 74 rules differently.
> - **PAGE-TYPES.md**: How to bias the rules for the **screen type** (dashboard / form / landing / detail / list / settings / onboarding). Read before building a specific page. Domain × page-type together = the actual design judgment.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bitjaru/styleseed](https://github.com/bitjaru/styleseed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
