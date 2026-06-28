---
trigger: always_on
description: This file is the runbook for an **AI coding agent** (and the human directing it) to fork this template into a new
---

# AGENTS.md: forking a new battle with an AI agent

This file is the runbook for an **AI coding agent** (and the human directing it) to fork this template into a new
historical battle. Read it fully before starting. The companion references are **PLAYBOOK.md** (the field-by-field data
schema) and **data.example.js** (a minimal working skeleton to copy).

---

## The one rule that overrides everything: do not fabricate history

This template renders a self-playing documentary. A documentary's worth is its **truth**. So:

- **Research and cite real sources first.** Before writing any data, gather the real date window, the opposing sides and
  their forces/commanders, the real geography (place names + coordinates), and the sequence of events, each tied to a
  source you can name.
- **Never invent a unit, a date, a movement, or a position to fill a gap.** If a fact is unknown or disputed, say so in
  `notes.caveats`. Do not paper over it with a plausible-sounding guess. A beautiful fabricated battle is worthless.
- The engine **enforces** this: `notes.sources` is a required field, and the boot validator refuses to start a battle
  without it. That is a floor, not a substitute for honest sourcing.

If you cannot source a battle to this standard, stop and tell the human. Do not proceed.

### Fictional battles: welcome, and held to the same standard

The engine renders fictional battles too (the bundled Example Battle is one), and a fiction does **not** break the rule
above. The line is simple: the rule forbids passing invention off as real history, not invention itself. A fictional
battle is fine as long as it is honest about being fictional and is built with the same care as a real one:

- **Declare it fictional.** Say so plainly in `notes` (the Example Battle's `notes.sources` reads "FICTIONAL
  DEMONSTRATION SCENARIO", which also satisfies the required non-empty `sources`). Never dress a fiction up as real
  history, and never attach a real battle's name, date, or place to invented events.
- **Stay true to the source material.** A fiction still has a source: a novel, a film, a game, an alternate-history
  premise, or its own established canon. Be accurate to THAT. This is fictional historical accuracy: period-plausible,
  internally consistent, and faithful to the world it depicts, with the same craft a real battle gets. A lazy,
  self-contradicting fiction is as worthless as a fabricated history.

So: real history is sourced and never invented; a fiction is marked as fiction and kept true to its own world. Both are
honest, and neither violates the rule.

---

## What a fork is

Edit only the **battle layer**; never the engine.

- **Edit:** `data.js` (the whole scenario), `flags.js` (each side's flag art), `index.html` (the `<title>` + social
  `og:` meta only: all on-screen chrome is data-driven), and the bounding box used by the tile fetcher.
- **Never touch (the engine):** `config.js`, `validate.js`, `app.js`, `core.js`, `projection.js`, `state.js`,
  `terrain.js`, `entities.js`, `director.js`, and the vendored libraries in `lib/`. They read every battle/faction/
  language value from `data.js`; editing them is almost always a sign you are doing something the data layer already
  supports. If you believe the engine genuinely cannot express your battle, raise it with the human rather than patching
  the engine.

---

## Procedure (in order)

1. **Research first.** Produce a sourced brief: date window, sides + forces + commanders, geography (named places with
   real lng/lat), the hour-by-hour or day-by-day sequence, and the source list. Do not start authoring until this exists.
2. **Set the map box.** Pick `meta.geo` (`minLng/maxLng/minLat/maxLat/Z`) covering the action, in `data.js`.
3. **Fetch the terrain + imagery tiles** for that box: `node tools/fetch_tiles.mjs` (cross-platform; add `--dry` to
   preview the tile count first). It reads the box from `meta.geo` — one source. Tiles come from two global, key-less
   providers, so any land region works; they are not committed (each fork fetches its own).
4. **Author `data.js`.** Copy `data.example.js` to `data.js` and replace its contents using your sourced brief. Follow
   the field schema in **PLAYBOOK.md**. The bilingual slots: `_zh` = your **primary / local language** (any script — set
   `meta.fonts` and `meta.dir:"rtl"` for non-Latin or right-to-left), `_en` = the **secondary** language (usually
   English). Units move along their `track` keyframes `{d,lng,lat,s,st}` — there is no flat position/strength field.
5. **Author `flags.js`.** Copy `flags.example.js` → `flags.js` — it ships reusable canvas primitives (`bands`, `disc`,
   `star`) and two worked example flags. Compose each faction's period-correct flag from the primitives; for richer real-flag painters (a period Union Flag, a
   16-ray Rising Sun, and more) see the Battle of Hong Kong repo's `flags.js` at
   https://github.com/keithligh/battle-of-hong-kong-1941/blob/main/flags.js.
   Use the correct historical flag for the period (a 1941 ensign, not the modern flag), and never a prohibited symbol.
6. **Edit `index.html`** — only the `<title>` and the `og:`/social meta (the page's head metadata). The on-screen title,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keithligh/cinematic-3d-battle-engine](https://github.com/keithligh/cinematic-3d-battle-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-28 -->
