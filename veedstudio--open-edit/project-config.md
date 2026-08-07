---
trigger: always_on
description: This repo turns a short video into ONE stylized **captioned MP4** — subtitles across every spoken beat,
---

# AGENTS.md

This repo turns a short video into ONE stylized **captioned MP4** — subtitles across every spoken beat,
rendered over the real footage by `veed-engine-cli` (the veed render engine). It runs via the `open-edit` skill
(`.claude/skills/open-edit/SKILL.md`) — the golden master. The default (recipe-backed) run is
**scripted end to end** — recipes are compiled code (`refs/html/<id>/recipe.ts`), zero tokens at run time;
subagents remain only for the opt-in vision analysis (run when the user asks to refine the style); the
CREATIVE path face-1 (the user brought their own reference/brand/concept) is authored INLINE by the
orchestrator, and creative iteration on a delivered result is REMIXED inline.

## To run a video
When the user says "run the open-edit skill on `<video.mp4>`", follow
`.claude/skills/open-edit/SKILL.md` exactly (the FAST PATH):
**preflight** (automatically prepare the workspace-local runtime; ask before global installs or updates) →
**prep** (VEED transcript via `veed/go.ts`, then `prep/prep.ts`: `meta.json` + `word-timings.json` from
VEED's real per-word times + base frames) → **sample ONE style** (`sample-style.ts`, facet-scored, seeded,
zero tokens → `style.json`; the pool is the recipes-only runtime index `refs/tags.json`, so the draw is
always recipe-backed) → **design + render** (`pipeline/scripts/generate-recipe.ts --run runs/<key>
--record`, a SCRIPT driving the full gate chain: the compiled recipe emits the .wv document → lint → `--verify`
with the mechanical ladder fix loop → records `out.silent.mp4` → probe-qa frame QA) → **mux audio**
(`pipeline/scripts/mux-audio.sh`). Deliverable → `runs/<key>/final/out.mp4`.
The **CREATIVE PASS** routes on the SHAPE of the ask (the user never learns recipes exist):
**face-1** — the prompt arrives WITH the user's own reference/brand/concept → the orchestrator authors
design+render INLINE per the brief (no subagent: nothing to orphan in headless runs, and the design stays
in the session's context so follow-up tweaks iterate instantly); the USER'S materials are the design
authority (they get looked at); 1-2 nearest recipe sheets ride along as craft
substrate (mechanics only) + a DIRECTION (content/mood/placement, never fonts/palette/device).
**face-2** — iteration on a delivered result: ANY creative input ("make it more interesting", a mood, an
aesthetic change) → REMIX, inline per the brief's REMIX MODE (donor sheets from the index, fresh
`runs/<key>-remix`, same gates); "show me more options / N versions" (zero creative input) → RE-ROLL
(N seeded draws through the script path); defect repairs → fix at the source + re-run the gates
(agent-authored templates are patched directly; recipe outputs are script-owned — regenerate via
inputs, the refine path, or a `--module` copy, never hand-edits).
The **analyse** step (nameless bg vision subagent → `analysis.json`) is OPT-IN: run it only when the user asks
to refine the style, then re-run design+render as the from-scratch inline pass (compiled recipes ignore
`analysis.json`; the pass composes from it) and re-mux — `analysis.json` existing IS the
fast-path/refinement switch.

## How the pieces relate
- `.claude/skills/open-edit/` — the orchestrator (the flow).
- `pipeline/director-brief.md` — the engine contract the from-scratch design pass obeys (the crown
  jewel); also carries REMIX MODE (the inline donor-blend contract) and the user-materials Method.
- `refs/html/<id>/recipe.ts` — the COMPILED recipes: one generator module per ref, next to its prose sheet,
  built on `pipeline/recipes/lib.ts` (the shared assembly rules); turns `meta.json` + `word-timings.json`
  into the final .wv document deterministically. Authored + validated offline (derived from the sheet);
  `hasRecipe` keys on the module existing.
- `pipeline/scripts/` — `sample-style.ts` (facet-scored seeded style pick) · `generate-recipe.ts` (runs the compiled recipe: generate → lint → `--verify` fix loop → `--record` → probe) · `lint-template.ts` (mechanical engine-limit gate) · `probe-qa.ts` (frame QA vs the source) · `resolve-video.ts` (the video-arg resolution rule shared by both entry points) · `synth-word-timings.ts` (imported by prep) · `extract-beat-frames.ts` (imported by prep) · `mux-audio.sh`, `preflight.sh`, `install-veed-engine.sh` (deterministic).
- `veed/` — VEED-native transcription + login (owns `transcript.json`; real per-word timings). `prep/prep.ts` — `meta.json` + `word-timings.json` + base frames (needs the VEED transcript). `refs/` — `html/` refs + `tags.json` (v3, the RUNTIME INDEX — recipes only, facet taxonomy, `fit` = aspect SOT) + per-ref `recipe.md` (the prose recipe sheet a compiled recipe is derived from — the fast path runs the compiled module, never the sheet; the creative pass reads sheets as craft substrate and REMIX donors).
- `config.ts` — all machine paths (ffmpeg / ffprobe / veed-engine). `docs/` — FLOW (orchestration) · recipe-format (the recipe law). Engine support matrix = the `feature-support.md` asset downloaded with the engine release into `.veed-engine/` (not vendored here).

## Hard rules (do not drift — these protect output quality)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veedstudio/open-edit](https://github.com/veedstudio/open-edit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-31 -->
