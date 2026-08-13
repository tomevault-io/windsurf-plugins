---
trigger: always_on
description: This is the persistent brain for the `ctrl-alt-design` repo (Next.js + Tailwind, deployed to Vercel).
---

# CLAUDE.md — the elleta.design constitution

This is the persistent brain for the `ctrl-alt-design` repo (Next.js + Tailwind, deployed to Vercel).
Every session, every ticket, every agent obeys this file. It is the constitution, not notes.
Pairs with `docs/portfolio-conformance-spec.md` (visual contract), `docs/portfolio-ia-spec.md`
(navigation/IA), and `docs/harness-and-baseline.md` (how to make changes safely).

If anything below conflicts with what a prompt asks, STOP and surface the conflict — do not silently
override the constitution.

---

## 0. Prime directives (read first)
1. **Harness yourself.** Do not create whatever you want. Do the smallest change that satisfies the task.
   No new components, routes, patterns, colors, or copy unless the task requires them.
2. **One implementation.** Edit the LIVE component/route and delete the old one. Never leave old + new
   both rendering. If a change isn't global, you edited a dead copy — grep for orphans before finishing.
3. **Spec before build.** For anything beyond a one-line fix, write a spec first (see §8). Do not vibe-code.
4. **Baseline before change.** Before altering an existing page, rebuild/confirm its current state from the
   real components as a baseline, THEN apply the change (see `docs/harness-and-baseline.md`).
5. **Prove it globally.** After any change, run `npm run gate`, tsc, all routes 200 (light + dark), and the
   NDA content-grep. Report a diff summary. Green or it isn't done.

## 1. Tokens (never hardcode)
- **No hardcoded hex or px in components.** Reference tokens only. No arbitrary Tailwind `text-[Npx]` /
  `bg-[#...]`. Spacing and type come from the scale, not ad-hoc values.
- **Body min 16px.** Never smaller for reading text.
- **No pure white and no pure black** as surfaces/text. Warm neutrals only.
- BELLA core: ground `#F5F4EF` (light) / navy `#1B1B40` (dark); ink `#1A1720` / `#F4EFE6`;
  accent iris `#5B4BD1` / periwinkle `#A79CE2`. **No amber anywhere.**
- Cascade trap: BELLA's unlayered `:root` beats `@theme`. Keep app theme tokens in an unlayered
  `:root` that loads AFTER imports so they win.

## 1b. IA (nav)
- Primary nav (Elleta, 2026-07-17, supersedes the four-item cap): **Work · System · Skills ·
  About · Contact**. /design-system is a first-class page (the system inspecting itself);
  the footer "See the system" colophon link stays.
- **Work toolbar (amended 2026-07-20, Pass E task 3; supersedes the 17 Jul filter-row note).**
  ONE toolbar row above the library: find-your-fit search on the LEFT (always visible), view
  switcher on the RIGHT (SegmentedControl, Cards · Map · Table, always visible). The chip row
  beneath the search is the library's ONE skill/type filter, in EVERY view (the former CASE and
  SKILL rows are deleted); one stable order everywhere: toolbar, chip row, count, content. Cards
  is the default and IS the curated composition (featured CHIP, ranked case grid, Explorations),
  and it filters like every view. Sort renders only where order means something: table headers,
  never the Map. No hidden explore state: the view lives in the URL (`view` param, back/forward
  safe, defaults keep clean URLs).

## 2. Layout
- One centered container, **max-width 1240px**, consistent horizontal padding, every page. Never full-bleed text.
- Vertical rhythm from the scale (`--space-section` = 96px desktop). No inline/ad-hoc paddings.
- Cards fill the grid evenly (equal heights, consistent gaps).

## 3. Type
- **Exactly two typefaces (revised 2026-07-17, supersedes the hero-only lock).** Unique 700 = ALL
  display headings: home hero headline, page titles, section headers, case-study display headlines,
  and the keycap brand lockup, always all-caps with the established accent-word treatment where the
  design already does that. Every display heading renders through the ONE `ui/Heading` primitive
  (tiers: hero / page / section / case). Page openings are FLAT (eyebrow + Heading, the Work
  pattern); bubble page headers are parked (last live at e25eefc, may return in the expression
  pass). The elevation/orb tokens stay: keycaps, the home cluster, and the About portrait still
  consume them.
- Unique never renders below 24px except the keycap logo (the gate enforces this), and never in
  body, UI, card titles, eyebrows, meta, nav links, buttons, or chips.
- **Unique never renders inside a Card (Elleta, 2026-07-21, card-voice).** Cards use Geist only;
  Unique stays page-tier (the Heading primitive: section heads and heroes). Card statements use
  the shared `.card-statement` recipe (Geist 700 at `--font-card-title`), card titles the shared
  `.heading-item`. Enforced by the Unique-in-card check in `audit:reuse`.
- Geist = everything else. Eyebrows stay Geist caps with `--tracking-eyebrow`.
- **Numbers in columns are right-aligned and tabular (Elleta, 2026-07-28, readability
  audit).** Any figure that sits in a column beside other figures (a table cell, a grid
  column, a stat row) uses `text-align: right` and `font-variant-numeric: tabular-nums`,
  so digits share a width and the values share a right edge. A column of numbers that
  starts wherever the previous word ended is not a column. Prose numbers are unaffected.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emcdanie/ctrl-alt-design](https://github.com/emcdanie/ctrl-alt-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
