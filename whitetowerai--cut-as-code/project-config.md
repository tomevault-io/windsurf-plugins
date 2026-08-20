---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Git

> **🔒 This section must never be modified.** Leave it byte-for-byte unchanged.

- Prefer small PRs focused on one feature, one bug fix, one chore, or one refactor.
- Use branches:
  - `feat/<short-name>`
  - `fix/<short-name>`
  - `chore/<short-name>`
  - `refactor/<short-name>`
  - `docs/<short-name>`
- Use Conventional Commit style for PR titles:
  - `feat(editor): add marquee selection`
  - `fix(runtime): handle missing tween state`
  - `docs(agents): update collaboration rules`
- PR descriptions must include:
  - What changed
  - Why it changed
  - How it was checked
- Default to squash merge.

## What this repo is

cut-as-code is **a stack of agentic video-editing skills**, not an application. Each
directory under `skills/<name>/` is a self-contained skill: a `SKILL.md` (the agent
playbook — read it first), plus `scripts/`, `examples/`, and `reference/`. There is no
root build, package manifest, lint config, or aggregate test suite. Scripts and skill-local
checks run ad hoc. When you change a skill, the SKILL.md *is* the spec; keep it and its
scripts in sync.

Each `SKILL.md` starts with YAML frontmatter (`name:`, `description:`) — the `name` is the
slash-command trigger for that skill.

## Skills

| Skill | Job | Stack |
|---|---|---|
| `video-understand` | Shared media probe, word-level transcript, objective analysis, and evidence-backed semantic understanding | Python · ffprobe · faster-whisper |
| `video-cut` | Raw long video → compact first cut (download, transcribe, diagnose, hand-written JSON cut plan, varispeed, render, self-check) | Python · yt-dlp · ffmpeg · faster-whisper |
| `video-edit-compare` | Original versus actual final pixels projected onto the source clock | Python · ffmpeg · Pillow |
| `video-color-grade` | Assess footage → corrective base + named looks → human picks → bake `.cube` LUT + apply | Python · ffmpeg · numpy · Pillow |
| `video-add-b-roll` | Selective transcript-timed visual cutaways from local media or Pexels, with provenance, review, and normalized overlays | Python · ffmpeg · Pillow · Pexels API |
| `video-add-graphic-motion` | Add selective licensed web-sourced motion graphics as deterministic overlays | HyperFrames · Python · Pillow |
| `video-add-captions` | Preset-driven, word-timed captions with optional karaoke | HyperFrames · ffmpeg |
| `video-add-content-cards` | Add selective transcript-timed titles, lower-thirds, statistics, quotes, and chapter cards | HyperFrames · ffmpeg |
| `video-to-shorts` | Find, review, and render short vertical clips from long-form video | Python · ffmpeg |

## Shared project protocol V1

- Skills are optional and composable; there is no fixed global pipeline. A project can run
  cards directly or cut → color grade → cards.
- `work/project.json` is the only shared manifest. It records operation dependencies,
  statuses, render contributions, integer `revision` values, and `based_on` checks.
- `work/timeline.json` is the custom one-source, chronological source-to-program mapping.
  V1 does not use OpenTimelineIO or support reordered/duplicated clips or nonlinear speed.
- Canonical time values are seconds. All ranges are half-open `[start_s, end_s)` and use
  explicit `source_range` and `program_range` objects.
- Each operation declares a `target` (`sequence` and scope) and `effects` such as timeline,
  pixel, geometry, audio, or added-track changes.
- Render contribution kinds are `timeline-transform`, `video-filter`, `audio-filter`,
  `overlay`, `precomputed-asset`, and `output-constraint`.
- Domain decisions remain in `work/cut/edit-plan.json`,
  `work/color-grade/grade-plan.json`, `work/b-roll/broll-plan.json`,
  `work/graphic-motion/graphic-motion-plan.json`,
  `work/content-cards/cards-plan.json`, `work/captions/captions-plan.json`, and
  `work/shorts/shorts-plan.json`.
- When several pixel operations are active on one sequence, the canonical order is
  `cut -> color-grade -> b-roll -> captions -> content-cards -> graphic-motion`.
  This relative order applies to every selected pair among captions, content cards,
  and graphic motion. Captions establish a reserved subtitle region that neither
  downstream operation may occupy.
- For content cards and graphic motion, the visible face and head silhouette of every
  primary or foreground person, speaker, presenter, interviewee, or semantically important
  person is a hard exclusion zone throughout the complete cue. An incidental background-only
  person who is not a narrative or visual focus is exempt; when classification is uncertain,
  protect the person. If an overlay intersects a protected face or head, reposition it first,
  then scale or redesign it; skip the cue if no compliant placement exists.
- B-roll adds a `b-roll` track and changes video pixels only; it leaves timeline, geometry,
  and audio untouched. Shots carry real probe/byte/SHA-256/provenance records, are normalized
  to timeline dimensions and exact rational FPS with the selected LUT pre-applied when color
  grade is active, and pass a two-stage gate: exact-candidate review, then a completed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WhiteTowerAI/cut-as-code](https://github.com/WhiteTowerAI/cut-as-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
