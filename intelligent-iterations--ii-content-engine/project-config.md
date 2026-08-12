---
trigger: always_on
description: This repo expects agents to turn user ideas into reusable templates, save the research and prompt contract behind those ideas, and run generation from saved artifacts rather than disposable prompts.
---

# AGENTS.md

## Repo Role

This repo expects agents to turn user ideas into reusable templates, save the research and prompt contract behind those ideas, and run generation from saved artifacts rather than disposable prompts.

Keep separate:
1. `template`
2. `research artifact`
3. `render settings`

Anything that goes into an AI prompt should come from saved repo state, not one-off inline prompt logic.

## Research

All research artifacts go under `research/`. This directory is gitignored — research is local working state, not committed to the repo. Use it for format research, prompt strategy notes, practitioner findings, and any intermediate research files. Reference research from templates and run artifacts, but keep the raw research in `research/`.

If the user provides substantial research directly in chat, such as a long pasted guide, strategy memo, or prompt reference they want used, treat that as valid research input. Distill it into saved repo state and rely on it by default instead of doing redundant external research that could overwrite or dilute the user's supplied direction. Only do additional research if the user asks for it or if there is a clear gap that blocks execution.

## Output Rules

- Video outputs go under `output/videos/<concept-slug>/`
- Carousel outputs go under `output/carousels/<concept-slug>/`
- Scheduled outputs go under `output/scheduled_videos/` or `output/scheduled_carousels/`

For video runs, save:
- `<slug>.md`
- `research.json`
- `<slug>_caption.txt`
- `asset-manifest.json`
- `plans/tool-plan.json`
- `plans/execution-plan.json`
- `clips/`
- `<slug>.mp4`

Do not scatter video artifacts across the repo root, `downloads/`, or ad hoc folders. A video run is complete only when its working files and final outputs live together inside that run's own folder.

## Required Rule Files

When the task matches, read and follow:

- Video work:
  - `.claude/rules/video-template-first.md`
  - `.codex/rules/video-template-first.md`
- Grok video/image prompting:
  - `.claude/rules/grok-video-prompting.md`
  - `.codex/rules/grok-video-prompting.md`
- Carousel work:
  - `.claude/rules/carousel-template-first.md`
  - `.codex/rules/carousel-template-first.md`
- Prompt best practices:
  - `docs/prompts/STORY_CHARACTER_PROMPT_GUIDE.md`

## Guidance Folders

Treat this folder as the source of truth for reusable documentation guidance:

- `docs/prompts/` for prompt-authoring guidance, asset-chain rules, and prompt best practices

Before authoring prompts, assets, or render jobs:

1. Check `docs/prompts/` for matching prompt guidance.
2. If the task is a continuity-sensitive character story, follow the workflow chain:
   `hero portrait -> derived character sheet -> scene start frames -> video`
3. For continuity-sensitive character stories, run the asset executor path before rendering clips.
4. For continuity-sensitive character stories, scene start frames should default to the approved ordered reference sheets for the visible characters. Do not auto-expand scene references back into sibling hero portraits unless the template explicitly needs that.

## Default Behavior

- New reusable video concepts should usually become templates.
- If the user request matches a saved prompt-best-practices guide, read the relevant file under `docs/prompts/` and apply it before authoring prompts or assets.
- Research is required for new or materially changed formats.
- Substantial user-supplied research can satisfy the research requirement for a format change if it is specific enough to drive the template or run artifact.
- For new or materially changed prompting work, do narrow research first, then wide research.
- Narrow research means searching for the exact format, subject type, and failure mode first, such as the specific character style, medium, or continuity problem you are trying to solve.
- Narrow research should start with likely creator language at the right level of abstraction, not abstract umbrella terms and not overfitted one-off phrasing. Search terms should usually be broad, highly relevant, and around 4 words or fewer.
- If the first narrow search pass is weak, reformulate and retry multiple times with adjacent niche phrasings, platform-native slang, and likely creator wording before concluding that the signal is missing.
- Do not stop after one weak search result set when the format is likely something practitioners have already explored. Try multiple query shapes across X, Reddit, search engines, and adjacent communities first.
- Do at least 5 distinct web research searches or passes before moving on from research into prompt writing for a new or materially changed format.
- Wide research means expanding from that niche into broader model tactics, continuity tactics, reference-image workflows, and adjacent successful formats.
- For prompting quality research, prioritize recent practitioner advice on X and Reddit over official docs. Use official docs mainly for capability limits, API behavior, and product mechanics.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [intelligent-iterations/ii-content-engine](https://github.com/intelligent-iterations/ii-content-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
