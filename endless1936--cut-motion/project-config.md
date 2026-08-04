---
trigger: always_on
description: cut-motion turns a talking-head video and an optional reference script into a tightly cut, motion-designed video. This repository is an Agent workflow, not a LangGraph, LlamaIndex, or fixed-template application.
---

# cut-motion Agent Protocol

cut-motion turns a talking-head video and an optional reference script into a tightly cut, motion-designed video. This repository is an Agent workflow, not a LangGraph, LlamaIndex, or fixed-template application.

`AGENTS.md` is the canonical operating contract. Codex, Claude Code, and compatible coding agents must follow it before editing media or authoring motion.

## Minimum input

- A local talking-head video.
- Optional preferences: caption mode, reference script, visual-axis strategy, output aspect, style references, and autonomy mode.

If media is missing, ask for its local path and stop. After receiving it, ask once for any known preferences without blocking progress when the user has none. A supplied script is a high-priority reference, never ground truth; the recording remains authoritative.

A reference script may contain local visual notes in full-width `【】`. These are medium-strength, non-exhaustive references for the immediately preceding semantic clause unless a note explicitly names another local range. They never enter released wording, replace whole-video MG and edit analysis, or count as final axis or motion approval. Ordinary `[]` remains spoken text. Reject empty, nested, unclosed, or unmatched `【】` during intake.

## Required outputs

Each run creates a job directory containing:

```text
jobs/<job-id>/
├── input/
├── state/
│   ├── project.json
│   ├── transcript.json
│   ├── transcript-reconciliation.json
│   ├── reference-script-annotations.json
│   ├── trim-plan.json
│   ├── design-system.json
│   ├── creative-confirmation.json
│   ├── workflow.json
│   ├── beat-map.json
│   ├── render-manifest.json
│   └── qa-report.json
├── roughcut/
│   └── a-roll.mp4
├── docs/
│   ├── caption-plan.md  # subtitles only
│   ├── creative-confirmation.md
│   └── motion-plan.md
├── captions/
├── hyperframes/
├── previews/
├── checkpoints/
├── logs/
└── output/
    └── final.mp4
```

Never overwrite the original source video. Every destructive-looking operation must produce a new artifact and update `state/project.json`.

Each job directory is an isolated working directory. Do not place job media, generated state, previews, or logs in the cut-motion repository root.

## Toolchain

Use the first available tool in each stage:

1. **Rough cut:** ChatCut project and editable timeline.
2. **Transcription:** recorded speech, supplied reference script, ChatCut transcription, then a local speech-to-text fallback.
3. **Precision trim:** FFmpeg and FFprobe.
4. **Motion design:** HyperFrames HTML/CSS with a single seek-safe GSAP timeline.
5. **Validation and render:** HyperFrames CLI, FFprobe, frame snapshots, and audio silence checks.

Remotion and Vibe Motion are not part of the default stack. Use them only when the user explicitly requests them and record the deviation in `state/project.json`.

## Operating modes

- `review` is the default. Always pause at the locked edit and final HyperFrames preview. Pause for creative confirmation and a short visual sample only when their documented trigger conditions apply.
- `auto` runs end to end using the gold-standard visual language and conservative trim thresholds. It still performs every quality gate.

Passing an automated check is not a substitute for a review gate in `review` mode.

## Workflow state machine

`state/workflow.json` is the authoritative state. Never advance by assumption or by merely creating the next artifact.

Use `scripts/workflow-state.mjs` for every transition:

```bash
node scripts/workflow-state.mjs jobs/<job-id>/state/workflow.json status
node scripts/workflow-state.mjs jobs/<job-id>/state/workflow.json advance --artifact <path> --note <summary>
node scripts/workflow-state.mjs jobs/<job-id>/state/workflow.json approve --actor user --note <feedback>
node scripts/workflow-state.mjs jobs/<job-id>/state/workflow.json revise --actor user --note <feedback>
node scripts/workflow-state.mjs jobs/<job-id>/state/workflow.json reopen <rough-cut|motion-plan|composition|delivery> --actor user --note <feedback>
```

The mandatory user review gates are:

1. rough-cut review;
2. final-preview review.

Creative-confirmation review is additionally required for any MG, `motion-copy`, B-axis or hybrid treatment, release-impact transcript ambiguity, or explicit user request. Visual-sample review is required for first or changed motion language, axis behavior, motion typography, high-attention MG, or an explicit caption-layout precheck. Caption-only subtitle work may proceed to final preview without a separate sample. Generate creative artifacts internally even when their user gate is skipped.

In `review` mode, stop at every required gate, present the artifact, and wait for the user to approve or request revision. In `auto` mode, required gates are recorded as `auto-approved`; inapplicable gates are recorded as `skipped`. The user may switch modes at any time.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Endless1936/cut-motion](https://github.com/Endless1936/cut-motion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
