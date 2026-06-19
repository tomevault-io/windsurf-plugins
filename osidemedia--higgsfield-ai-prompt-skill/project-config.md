---
trigger: always_on
description: >
---


# Higgsfield AI Prompt Skill

**Language rule:** Reply in whatever language the user writes in.

---

## HARD RULES — pre-delivery checklist

These rules apply to every Higgsfield response. They are written as a pre-delivery checklist the agent runs *before* sending the response, not as prohibitions stated and then forgotten. The failure mode they prevent is **plausibility-over-verification** — producing a response that looks correct because the agent's training data knows the rough shape of Higgsfield work, rather than because the agent actually read the skill files and verified the platform's ground truth.

**Before delivering any Higgsfield response, confirm in this order:**

1. **Routing line present.** First line of response names which sub-skills you routed to (e.g. "Routing to higgsfield-prompt + higgsfield-camera for an Atmosphere push-in"). One line, then the work. Missing routing line = response is incomplete; add it.

2. **Routed sub-skills opened and read in this conversation.** Match the user's ask to the routing table below, open the matching sub-skill files with the read tool, and READ them. Root `SKILL.md` and `skills/higgsfield-prompt/SKILL.md` are mandatory at minimum on any prompt request. Grepped snippets do not satisfy this rule. Full reads do. If your only access to root `SKILL.md` or `skills/higgsfield-prompt/SKILL.md` in this conversation came from grep results, you have not satisfied this rule — open the file. Platform vocabulary, preset names, and model parameters must come from the files because this platform's lineup changes between releases.

3. **Named vocabulary verified, not invented.** Camera preset names, motion preset names, model names, CLI flag forms, and MCP tool parameter names all come from the skill files or from verification. For model parameters, enums, and durations, verify against `specs/model-specs.yaml` first — it is generated from a dated `models_explore` snapshot (see `snapshot_date` inside the file); if the snapshot is stale (>30 days), verify live instead (`higgsfield model get <model>` for CLI param schemas; `models_explore` for MCP). If you found yourself thinking "this flag probably looks like X" or "this preset is probably called Y" — stop. Read the file or run the verification command. Plausibility is not validity. Do not substitute generic video-prompt vocabulary for named Higgsfield presets; do not invent model versions, camera presets, or motion-preset names. If the user names one you don't see in the skill files, say so and ask for clarification.

4. **MCSLA structure intact on video prompts.** Model · Camera · Subject · Look · Action. Five layers, every video prompt, unless the user explicitly opted out.

5. **Shared negative constraints appended.** Pull positive-phrasing prevention phrases from `skills/shared/negative-constraints.md`. Do not paraphrase from training; use the exact phrasing from the file. (Kling 3.0 prefers positive phrasing over negations; using negation-form constraints when the file says positive is a fidelity miss.)

6. **Preflight surfaced when applicable.** If execution intent is signaled (CLI / MCP / bundled-skills mentioned) AND a video-class or high-cost model is named OR a budget concern is named, surface the two-step preflight (`model get` / `models_explore` for schema, then cost estimate). See `skills/higgsfield-stack/SKILL.md` § Preflight discipline.

7. **Aspect ratio is an enum, not a free-form value.** Check the model's allowed ratios against `specs/model-specs.yaml` before writing them into the header; if the snapshot is stale (>30 days), verify live via schema (`models_explore` / `model get`). Example of why this matters: Seedance 2.0 supports native 21:9, Kling 3.0 does not. Anamorphic / 2.35:1 / 2.39:1 are *style register* vocabulary for the Look line, not output ratios. See `vocab.md` § Aspect Ratio: output spec vs. style register.

8. **Prompt under 200 words.** Soft cap from MCSLA section. Going over is a signal you're padding rather than locking — tighten.

**If any of items 1–8 are missing or unverified, the response is incomplete. Complete them before sending, not after.**

---

## What Is Higgsfield?

Higgsfield is a cinematic AI video and image generation platform built for filmmakers and
creators. Unlike single-model tools, Higgsfield hosts **multiple generation engines** on one
platform — Kling 3.0/3.0 Omni/3.0 Motion Control, Sora 2, Google Veo 3.1/3.1 Lite, Wan 2.7/2.6/2.5,
Seedance 2.0/Pro, Minimax Hailuo 2.3/02, Higgsfield DoP (Lite/Standard/Turbo) for video; Soul 2.0, Soul Cinema Preview,
Soul Cast, Nano Banana Pro/2, Kling Image 3.0/Omni, Seedream 4.0, GPT Image 2.0,
Flux 2/Kontext for images — plus a library of 100+ named **Motion Presets**, a **Soul ID**
character consistency system, **Cinema Studio 2.5**, **Cinema Studio 3.0** (Business/Team plan), and **Cinema Studio 3.5** with Soul Cast AI actors, native dual-channel stereo audio, and 80+
one-click **Apps**.

---

## Working Folders — file handling

The project has a `workspace/` folder with three subfolders. Use them for every
task that involves a user-provided document or a file you produce. This keeps
uploads and deliverables out of the project root.

| Folder | Role | Your behavior |
|--------|------|---------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OSideMedia/higgsfield-ai-prompt-skill](https://github.com/OSideMedia/higgsfield-ai-prompt-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
