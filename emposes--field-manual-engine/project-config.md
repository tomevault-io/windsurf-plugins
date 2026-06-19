---
trigger: always_on
description: Generate a complete interactive field-manual website about ANY topic (espresso, options trading, Roman history, music theory, …). Trigger on "interactive manual", "field manual", "interactive guide about X", "interactive encyclopedia about X", "interactive handbook about X", "teach X as an interactive site", or "build me a manual/encyclopedia like the AI Encyclopedia". Produces a Palantir-dark × Apple-restraint static multi-page site with live instruments, hover glossary, optional KaTeX math and
---


# THE FIELD MANUAL ENGINE

You are about to build an interactive field manual that is **indistinguishable in
craft from the AI Encyclopedia** (llm-manual.vercel.app) — about whatever topic the
user names. The engine is topic-agnostic; the craft rules are not negotiable.

Before doing anything else, read these files from this skill's directory:

1. `engine/AUTHORING.md` — the per-chapter spec (components, voice, self-checks)
2. `engine/templates/chapter-template.html` — the chapter skeleton
3. `engine/templates/index-template.html` — the hub skeleton
4. `engine/assets/js/GLOSSARY-NOTE.md` — how to swap the glossary per topic

Skim `engine/assets/css/manual.css` (the design system) and `engine/assets/js/shared.js`
(the FM helper API) so you know what exists before inventing anything.

---

## A · INTAKE — ask before you build

Ask the user these, compactly, in one message. If they say "just build it," use the
defaults in brackets and proceed without further questions.

1. **Topic & scope edge** — what's in, what's out? ("espresso" = the drink + dialing
   in + machines? or also roasting and café business?) [you draw the line, state it]
2. **Audience & depth** — curious beginner, serious hobbyist, working practitioner,
   or a mix? [mix: arc starts INTRO, ends ADVANCED]
3. **Chapter count** — 6–12 chapters + capstone. [propose 8 + capstone]
4. **Accent preset** — show the five pairs from section H, recommend the one that
   fits the topic's temperament. [your recommendation]
5. **Deploy target** — Vercel, GitHub Pages, or just local? [local + ready-to-deploy]

Then, **before writing any HTML**, post a one-screen plan and get a nod:
the chapter arc (titles + one-line theses + level badges), the topic's
quantities-with-knobs and process-worth-simulating (section C), and the accent
choice. This plan is cheap; rebuilt chapters are not.

---

## B · THE PEDAGOGY ARC

Every manual follows the same five-act arc, scaled to the chapter count. Each
chapter gets a difficulty badge: **INTRO** (no prerequisites beyond curiosity),
**CORE** (comfortable with the manual's notation/canon), **ADVANCED**
(practitioner-adjacent).

| Act | Share | What it does | Badge drift |
|---|---|---|---|
| **Foundations** | ~25% | The atoms of the domain: definitions, units, the one core loop or object everything else builds on. Reader leaves able to *name things precisely*. | INTRO |
| **Mechanics** | ~25% | How the atoms interact — the central mechanism opened up and played with. The manual's densest instruments live here. | INTRO → CORE |
| **Systems** | ~25% | Composition at scale: workflows, trade-offs, failure modes, "the bill." Where the calculators and trade-off frontiers live. | CORE |
| **Frontier** | ~15% | Live debates, recent developments, what experts disagree about, what's unsolved. Honest hedging is content here, not weakness. | ADVANCED |
| **Capstone** | 1 chapter | A synthesis instrument: the reader *configures something end-to-end* with live numbers (a dossier/result card), plus 3–5 drill cards that grade instantly. | CORE |

Example mappings (so this never collapses into vagueness):

- **Espresso (8+1):** what coffee is → grind & water chemistry → the shot
  (pressure/flow/extraction) → dialing in → milk → machines → sourcing & roast
  levels → the frontier (turbo shots, profiling debates) → capstone: design your
  bar + diagnose five broken shots.
- **Options trading (10+1):** contracts & payoffs → pricing intuition →
  Black–Scholes & the Greeks → volatility → spreads → income structures → risk &
  sizing → market microstructure → frontier (0DTE, vol products) → capstone:
  build a position, watch it age through a vol shock.
- **Roman history (9+1):** geography & sources → kingdom → republic machinery →
  the army → the revolution century → principate → economy & daily life → crisis
  & dominate → fall debates → capstone: the empire timeline scrubber + drills.

Chapter ordering rule: each chapter's hero lists `BUILDS ON` — the arc must be a
DAG that never points forward.

---

## C · THE INSTRUMENT ARCHETYPE LIBRARY

**This is the heart of topic-agnosticism.** Before scaffolding, do the mapping:

> **List the topic's 3–5 quantities-with-knobs** — continuous variables an expert
> actually reasons over (espresso: dose, grind setting, water temp, pressure, time;
> options: strike, days-to-expiry, implied vol, rate; Rome: year, army size, grain
> supply; music theory: tempo, interval, scale degree, voice count).
> **And ONE process-worth-simulating** — the multi-step temporal thing at the
> domain's center (the 30-second shot; an option's life through theta decay; a
> campaign season; a 12-bar progression resolving).

Then assign archetypes: quantity-clusters → 1/3/7/8, the process → 2/5, structure →
4/9, judgment → 6, retention → 10. Every chapter gets ≥2 instruments from the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Emposes/field-manual-engine](https://github.com/Emposes/field-manual-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
