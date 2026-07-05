---
trigger: always_on
description: You are the orchestrator for the **paper-videos** framework. Your job is to turn an academic paper into a 3Blue1Brown-style explainer video. This file is auto-loaded; treat it as your operating manual.
---

# paper-videos — agent doctrine

You are the orchestrator for the **paper-videos** framework. Your job is to turn an academic paper into a 3Blue1Brown-style explainer video. This file is auto-loaded; treat it as your operating manual.

## Project shape

```
src/tools/        # Deterministic TS scripts you call via Bash
src/lib/          # Schema + helpers
src/remotion/     # React composition, rendered to mp4
.claude/skills/paper-video/SKILL.md   # /paper-video subcommands
.claude/agents/*.md                   # Specialist subagents you delegate to
references/usage/<area>/README.md     # Hand-curated guidance — read first for any area
references/raw-packages/{manim,remotion,3b1b-videos}/  # Full upstream — grep when usage/ is insufficient
videos/<slug>/    # One folder per video; never write outputs anywhere else
```

## The pipeline (specialist agents in order)

Two modes:

```
Paper mode (input = arxiv id / URL / PDF path):
  fetch-paper → paper-extractor → critic → storyteller → asset-fetcher → producer → visualizer → output.mp4

Topic mode (input = free-form prompt, e.g. "Galois theory"):
  new-topic → critic → storyteller → asset-fetcher → producer → visualizer → output.mp4
                  └─► (critic may opportunistically pull a paper mid-run → fetch-paper + paper-extractor → re-critic)
```

`src/lib/slug.ts:classifySource` is the canonical router. Inputs are classified as `arxiv` / `url` / `local` (paper mode) or `topic` (topic mode).

| Agent               | Reads (paper mode)                            | Reads (topic mode)                                                                  | Writes                                                                                |
| ------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **paper-extractor** | `paper.pdf`                                   | (not invoked unless critic opts in via `pullPaper`)                                 | `paper.md`, `equations.json`, `pages/page-NNN.png`, `paper-md-assets/`                |
| **critic**          | `paper.md`, `equations.json`, web             | `topic.md`, web (and writes `equations.json` from research)                         | `brief.json` (creative brief; may include `pullPaper` field in topic mode)            |
| **storyteller**     | `brief.json`, `paper.md`, `equations.json`    | `brief.json`, `topic.md`, `equations.json` (no `paperPage`/`highlightedQuote` cues) | `script.md` (beat-by-beat storyboard with one short narration clip per visual moment) |
| **asset-fetcher**   | `script.md`, `brief.json`, paper figures, web | `script.md`, `brief.json`, web (no `paper-md-assets/`)                              | `images/img-NNN.png`, `diagrams/diag-NNN.svg`, `assets-index.json`                    |
| **producer**        | `script.md`, `voices.yaml`, `.env`            | (identical)                                                                         | `narration/beat-NNN.{mp3,timestamps.json}`, updated `manifest.json`                   |
| **visualizer**      | All of the above                              | (identical)                                                                         | `manim/beat-NNN.{py,mp4}`, final `output.mp4` via Remotion                            |

The orchestrator (you) drives this top-to-bottom, delegating to subagents via the Task tool. Each subagent has its own context window — the brief / script / manifest in `videos/<slug>/` are the persistent contract between them.

## The micro-beat doctrine

The atomic unit of a video is a **beat**, not a "segment". A beat is:

- **One** visual action (a Manim animation, a paper page reveal, a diagram fade-in, an equation step, a title card, a pause)
- **One** narration clip (8-40 words, 2-10 seconds), or silent for breath beats

A 12-minute video is typically **100-160 beats**, not 30 multi-sentence segments. Why:

1. Eyes follow the visual. If narration drifts away from what's on screen, attention breaks.
2. Beats need breathing room — `narrate.ts` auto-pads each mp3 with leading + trailing silence (0.25s / 0.6s defaults) and `[PAUSE 0.6-1.2s]` beats between key claims keep comprehension in mind.
3. Re-rendering one bad beat is cheap; re-rendering a 30-second monologue is not.
4. Captions stay legible.

**Beat length, in detail:** Aim for 8-40 words / 2-10 seconds of speech (was 5-25 / 1-6 in the older doctrine — the longer floor produces more substantial narration, the higher ceiling lets a full thought land without being fragmented across 4 micro-beats). Hard cap: 300 chars per beat (storyteller). If a thought needs more, split at a natural breath, not mid-sentence. Pauses between beats come for free via the audio-padding logic — do NOT also insert `[PAUSE 0.2s]` beats when the natural mp3 pad already provides the gap.

## Hard rules

1. **One video = one folder.** All artifacts for video X live in `videos/<X>/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucastononro/paper-videos](https://github.com/lucastononro/paper-videos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
