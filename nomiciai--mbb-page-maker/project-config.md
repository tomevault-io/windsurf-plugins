---
trigger: always_on
description: Create, edit, package, or export consulting-style HTML slide decks for strategy, board, investment, pitch, and executive presentations. Use when the user asks for an HTML PPT, slide deck, pitch deck, investment memo deck, board report, static browser-openable presentation, WeChat-shareable PDF/PNG previews, or updates to an existing MBB Page Maker deck.
---


# MBB Page Maker

This AgentSkill produces static HTML presentation decks with consulting-grade page logic: clear titles, tight storylines, exhibit-first layouts, and executive-ready visual hierarchy.

The source deck must run without a build step: plain HTML, CSS, and JavaScript. Source decks use predownloaded local fonts through `assets/css/fonts.css`. The delivered package must be self-contained: no remote runtime scripts, remote images, remote font URLs, stylesheet imports, or non-embedded media.

## Authoring Philosophy

Treat every deck as a decision tool, not a styled page collection. The agent owns the authoring judgment: infer the audience, decision, storyline, layout, component, and theme from the user's material instead of asking the user to design the deck.

Hard rules:

- Start decision-backwards: audience, decision, answer, evidence, implication, action.
- Keep source fidelity: use user material, marked assumptions, or approved external data; never invent evidence.
- Let evidence shape drive components; if evidence is sparse, use qualitative pages and visible assumptions.
- Use answer-first titles and visuals only when they support the argument.
- Treat archetypes and showcases as references, not templates; complete the deck and exports when material is sufficient.

## Install

Use the copyable terminal command:

```bash
npx skills add https://github.com/NomiciAI/mbb-page-maker
```

Repository: [NomiciAI/mbb-page-maker](https://github.com/NomiciAI/mbb-page-maker)

If an agent reports "Unknown skill" after installation, read `references/agent-compatibility.md`. Some clients install the canonical package under `.agents/skills/` but require a symlink or copy under their own skill directory.

## Golden Path

For real deck work, follow this path in order:

1. Read `references/authoring-guide.md` before creating or materially rewriting a deck.
2. For strategy, board, investment, pitch, transformation, or full-deck work, read `references/consulting-thinking.md`.
3. Infer audience, decision, likely answer, constraints, and external-data permission.
4. Extract evidence: claims, numbers, comparisons, timing, risks, decisions, assumptions, and gaps.
5. Choose references only as needed: `references/full-decks.md` for full storylines, `references/content-to-exhibit-router.md` for evidence-to-component routing, `templates/showcase/README.md` for page patterns, `references/visual-assets.md` for visual support.
6. Start from `templates/starter-deck.html` or the user's existing deck. Do not start from a blank file.
7. Create a compact internal slide plan: message, evidence source, evidence shape, selected component, selected showcase pattern when useful, fallback, and output role.
8. Assemble slides using existing CSS layers, layout shells, components, and theme tokens.
9. Run `scripts/check-deck-quality.sh path/to/index.html` and `scripts/check-deck-contrast.sh path/to/index.html`; fix failures.
10. Run `scripts/render.sh path/to/index.html` unless the user explicitly asks for source HTML only.
11. Deliver `dist/package/index.html`, `dist/package/index.pdf`, `dist/index.pdf`, and `dist/png/` as the default share set.

If any export step fails, fix the source deck or dependency problem and rerun the same command. Do not hand-wave missing package assets, PDF, PNGs, or audit failures.

## Current Scope

This is the foundational HTML skeleton phase. Use the starter deck as the base page system, then expand case-by-case references later.

- Use `templates/starter-deck.html` as the minimal HTML PPT starting point.
- Use `templates/deck.html` only as the design-system gallery and review tour.
- Use `templates/full-decks/*/README.md` as the fast indexing layer, then `templates/full-decks/*/index.html` as first-class deck archetypes for storyline pacing, page roles, density, component coverage, and export packaging. Do not copy their page order or content verbatim into user decks.
- Treat `templates/full-decks/*/index.html` as agent-facing archetype references. `examples/*` are independent public demo decks; do not use examples as the authoring source unless explicitly recovering newer content.
- Use `templates/showcase/README.md` and `templates/showcase/*.html` for reusable page-level thinking patterns and theme + layout + component combinations, not full storyline templates.
- Use the CSS layers in order: `fonts.css`, `base.css`, `layouts.css`, `components.css`, `illustrations.css`, then one file from `assets/themes/`.
- Use `assets/js/runtime.js` for keyboard navigation and print/export mode.
- Read `references/authoring-guide.md` before creating a real deck.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NomiciAI/mbb-page-maker](https://github.com/NomiciAI/mbb-page-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
