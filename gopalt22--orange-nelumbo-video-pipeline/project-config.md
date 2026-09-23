---
trigger: always_on
description: Visualization-heavy explainer videos for JEE PYQs, built with Manim CE + edge-tts,
---

# Orange Nelumbo — JEE PYQ Video Pipeline

Visualization-heavy explainer videos for JEE PYQs, built with Manim CE + edge-tts,
rendered locally. Company moat: interactive, visual learning — never a talking
equation dump.

## Pipeline (agents run IN SERIES per question)

```
questions/<qid>/question.md   (the PYQ only — the pipeline SOLVES it)
   │ 1. question-analyst  → analysis.json   (solves + code-verifies the answer)
   │ 2. storyboard-writer → script.md (founder-reviewable script) + storyboard.json
   │ 3. python -m core.tts questions/<qid>  → audio/*.mp3 + timings.json
   │ 4. manim-coder       → scene.py  (Boundary-law trained: right first time)
   │ 5. python pipeline/build.py questions/<qid> --quality k --parallel 4
   │       → out/final.mp4   (the ONLY render; no loops, no review stages)
```
(qc-reviewer agent exists for optional post-mortems only.)

Input contract: the team supplies the PYQ *and its solution* in question.md.
The pipeline's value-add is the script + on-screen visual design, not solving.
Target video length: **3-5 minutes** (~1800-2800 narration chars at our
measured TTS rate of ~10 chars/s); the visualization section carries ~30-40%
of runtime.

To make a video for a new question: create `questions/<qid>/question.md` with
the verbatim PYQ + exam/year + solution, then run the agents above in order
(Agent tool, subagent_type = the stage name). Steps 3/5 are plain scripts.

ONE-SHOT PRODUCTION (IMPORTANT): there are NO QC loops and NO iteration
renders. Agents must NEVER rasterise frames — not stills, not `-ql` smoke
renders, not partial `-n a,b` checks. The only command an agent may run is
the zero-frame `manim --dry_run`; the single production render at the end is
the first and only time pixels are drawn. The layout discipline is trained into the agents (see
.claude/agents/manim-coder.md "Boundary law") so the scene is right the
first time. Per question, exactly ONE command produces the video:
`build.py --quality k --parallel 4` — it runs a ~2-min zero-frame sanity
pass internally (aborts before wasting a 4K render if the scene is broken),
then renders across 4 concurrent manim processes, concatenates, muxes.
`--quality qc` exists only for post-mortem debugging, never in the flow.
The pipeline is FULLY AUTOMATED end-to-end: no human review gates. script.md
is still written (audit trail + optimization), but the pipeline proceeds
straight to TTS, scene coding, QC and render. The qc-reviewer's SHIP/FIX
verdict is the only gate: FIX loops back to manim-coder automatically until
the video passes.

## Screen layout (founder spec 2026-07-20; enforced by core/layout.py + runtime QC)

**Question phase (full-frame, before solving):**
- The WHOLE question, verbatim (never shortened), center-aligned in the
  upper middle (`layout.QUESTION_TEXT`).
- Below it: the diagram, a bit smaller, on the LEFT (`QUESTION_FIGURE`);
  ALL options beside it on the right (`QUESTION_OPTIONS`), all white
  (`show_options(..., zone=layout.QUESTION_OPTIONS)`).

**Solution mode** (boots after extraction via `begin_solution_mode()`:
each layer gets a soft rounded panel — graphite surface step, faint warm
border, easy on the eyes — drawn automatically by JEEScene; scenes never
draw their own zone borders):
- **Top strip** = GIVEN ribbon: extracted & derived facts as chips.
- **Left + center** = the WORK AREA: visualization and calculation SHARE it
  (one panel, one sliding divider). `set_split("viz"|"balanced"|"calc")`
  re-balances as the explanation shifts; place content via
  `place_viz()`/`place_calc()`. The diagram grows, shrinks and steps aside
  as needed — but the calculation is ALWAYS on screen (animated, synced to
  narration, terms pulsing) and the diagram never fully vanishes. A dead
  half-screen kills attention; the old fixed calc column sat empty ~70% of
  the video, which is exactly what this replaces.
- **Right column, top half** = `EQUATION_PANEL`: equations & concepts used.
- **Right column, bottom half** = `OPTIONS_PANEL`: option rows that turn
  green/red via `focus_option`/`verdict_option` (chevron -> derive ->
  compare, SFX built in).
- **Bottom 13%** = subtitle band: 2 lines max, small font (17pt), karaoke
  highlighting. Nothing else may ever enter it.
- **Margins**: elements from different groups keep clear of each other —
  runtime QC flags text-vs-text AND text-vs-opaque-box overlaps (0.08 units
  minimum penetration). Boxes are always sized FROM their text, never fixed.

## Brand rules

- COLOURS v2 (founder map 2026-07-22, `core/chapter_colors.py`): the video
  background IS the subject — a dark→colour gradient drawn automatically by
  JEEScene. Physics=BLUE, Chemistry=PINK, Maths=GREEN, 20 chapter shades
  each; every chapter also gets its own orange, gold and shade-of-white.
  Scene code uses `self.C["bright"|"tint"|"dark"|"orange"|"gold"|"white"]`.
  The 7 semantic hues (`brand.COLORS`: signal cyan givens, ignition active,
  titanium scaffold, correct/error/amber) stay FIXED whenever real work is
  on screen. Full rules: `brand/SUBJECT_PALETTES.md`.
- SHAPE: sharp corners everywhere — `corner_radius=brand.CORNER` (0.02),
  never a literal. Rounded boxes read soft; the brand is precise.
- Brand fonts (TTFs in brand/fonts, auto-registered by core.brand):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gopalt22/orange-nelumbo-video-pipeline](https://github.com/gopalt22/orange-nelumbo-video-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
