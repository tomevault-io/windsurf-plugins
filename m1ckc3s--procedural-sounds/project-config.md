---
trigger: always_on
description: A generate-first web app for procedural UI sounds (taps, hovers, transitions, success,
---

# CLAUDE.md

## Project

A generate-first web app for procedural UI sounds (taps, hovers, transitions, success,
error, warning, notification): pick a category, generate until one fits, play it in
context, export it. Every sound is synthesized live from a recipe; there are no audio
files in the product. The refinement loop is regenerate, not fine-tune.

The sound library and the training data are the same asset: a human curates generated
candidates, and those verdicts train the generators. See HOW-IT-WORKS.md.

## Docs map (load as needed, don't preload)

Committed docs live in `docs/`, with `README.md`, `CONTRIBUTING.md`, `LICENSE` and
`THIRD-PARTY-NOTICES.md` at the repo root. This file lives in `.claude/`.

- [docs/GETTING-STARTED.md](../docs/GETTING-STARTED.md) - install, run, the two surfaces, and what each workbench tab is for.
- [docs/HOW-IT-WORKS.md](../docs/HOW-IT-WORKS.md) - the explainer: what each engine does, how the learning works, and a glossary for every internal term. Read this first if you are new.
- [docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md) - what is built right now: repo map plus settled decisions. Update it when a feature LANDS. Never business or undecided content, never pack/sound counts (the data is the only truth).
- [docs/TRAINING.md](../docs/TRAINING.md) - what each workbench tab's keep and delete actually write, and the curation working guide.
- [docs/TODO.md](../docs/TODO.md) - live actionable work ONLY.
- [THIRD-PARTY-NOTICES.md](../THIRD-PARTY-NOTICES.md) (repo root) - the ONE attribution and licensing source of truth. A new source gets a row there before its code or data lands.

### If the answer is not in the docs above, GO LOOK. Do not answer from nothing.

The committed docs are a summary, not the whole record. Before saying "there is no X" or
"that does not exist", search these, in this order. This is not optional: several
confidently-wrong answers in this project came from an agent describing the docs instead of
reading the repo.

1. **`docs-local/`** (gitignored, not published) - project history, the done log, the
   engine, positioning material, superseded design notes, and
   full transcripts of past working sessions. The reasoning behind settled decisions
   usually lives here and nowhere else. `grep -ri "<term>" docs-local/` first.
2. **`data/reference/`** - the imported seed packs and `UPSTREAM-LICENSE`. The upstream
   design-rule skill and the captured reference playground that used to sit here are in
   `docs-local/` now (calibration reference only, never a runtime feature).
3. **`data/pool/*.json`** - the curation state IS the truth for anything countable. Never
   guess a count; read the file or the workbench chips.
4. **The code.** Trace the call sites. A claim about what runs at generation or play time
   is verifiable in minutes and should never be asserted without doing so.

Nothing in `docs-local/` is instructions or current state. If a fact in there turns out to
be load-bearing, promote it into a committed doc rather than linking to it.

## Commands

`npm run dev` (dev server), `npm run build` (production build), `npm run lint` (ESLint).
The workbench only works under `npm run dev`: its API routes write curation state to
`data/pool/*.json` on the local filesystem.

## Where things live

- `lib/audio/` - the synth core and the generators. `patch.ts` (the shared `Patch` type), `synth.ts` (the recipe player), `randomize.ts` (pool building + the frozen variation pass), `create.ts` (the creator behind v2), `compose.ts` (category grammars and archetypes), `invent.ts` (the Invent draw; dice keys `g:*` and `hybrid` in `invent-feedback.json`; its code key is still `nebula`), `wild.ts` (the untrained discovery paths behind Wild), `taste.ts` (feature buckets + deleted-twin fingerprints), `gates.ts` (category gates), `limits.ts` (ear-safety clamps), `loudness.ts` (play-time leveling; never rewrites a patch), `offline.ts` (offline render + peak/RMS measurement), `similarity.ts`, `invert.ts`, `categories.ts`, `context.ts`, `effects.ts`, `atlas.ts` (the vocabulary descriptions the atlas page renders). The instrument-first engine: `instruments.ts` (43 coherent single-note voices), `figures.ts` (gesture shapes plus the four spaces), `craft.ts` (the per-category caster, and `castFrom` underneath it), `prospect.ts` (five engines behind one button).
- `app/page.tsx` - the product. Eight source buttons (the seven categories plus one experimental), a Familiar/Exotic toggle (v1/v2 internally), and the `SoundStage`. The four-stop slider is GONE: only the two curated stops ship, and the experimental button draws from `prospect.ts`.
- `app/workbench/` - the curation tool. One page driven by `?tab=`, in a sidebar shell, plus four real routes. In sidebar order (`components/workbench/nav.tsx` is the truth). Sounds: Library (slug `review`), Variations, Creations, Prospect (`/workbench/prospect`), Craft (`/workbench/craft`), Invent, Wild. Tools: Atlas (`/workbench/atlas`), Editor, Import (`/workbench/import`, paste a copied sound back into the library), Dedupe, Calibrate, Trash (the only place a delete can be undone). `app/api/*` is file-backed persistence for `data/pool/*.json`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m1ckc3s/procedural-sounds](https://github.com/m1ckc3s/procedural-sounds) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
