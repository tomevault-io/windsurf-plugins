---
trigger: always_on
description: This folder is a Higgsfield-first AI video production workspace. When the user opens this folder, they are starting (or continuing) a high-end AI video project — character builds, scene plates, Seedance video shots, optional music + title card, final upscale.
---

# AI Video Making — Workspace Guide for Claude

This folder is a Higgsfield-first AI video production workspace. When the user opens this folder, they are starting (or continuing) a high-end AI video project — character builds, scene plates, Seedance video shots, optional music + title card, final upscale.

## How this repo activates — two paths

This repo is designed to run in **either** of two Claude environments. The behavior differs slightly depending on which one you (the Claude agent reading this) are running in:

### If you're running in Claude Code CLI (recommended path for users)

**You do not need to tell the user to upload anything to claude.ai.** The four `SKILL.md` files (`banana-pro-director/SKILL.md`, `cinema-worldbuilder/SKILL.md`, `ai-film-director/SKILL.md`, `video-qa/SKILL.md`) are right here in the repo — read them directly with the `Read` tool when the matching trigger fires, and follow them as your operating instructions:

- User says *"I want to make an AI video / music video / commercial / short film / spot"* → read `ai-film-director/SKILL.md` and follow it as the orchestrator.
- User asks for an **image prompt** (character sheet, environment plate, prop sheet, GPT-2 detail shot) → read `banana-pro-director/SKILL.md` and follow it.
- User asks for a **Seedance video prompt** (any single shot or sequence) → read `cinema-worldbuilder/SKILL.md` and follow it.
- User says *"QA my project / check for drift / is this ready to ship"* → read `video-qa/SKILL.md` and follow it.

When the orchestrator instructs you to "call cinema-worldbuilder" or "hand off to banana-pro-director," that means: read the relevant specialist `SKILL.md` and adopt its prompt-grammar rules for the next output. You are the one agent — there is no separate skill invocation step in Claude Code.

In Claude Code you also get **filesystem-native reference handling** (no upload — Read tool picks up images directly), **can run `./scripts/new-project.sh` natively**, and **can install `ffmpeg` in-session** when QC needs it. Use these advantages.

### If you're running in claude.ai (web / desktop)

The four skills exist as installed claude.ai skills (Settings → Capabilities → Skills, uploaded as `.zip` files). They auto-activate based on conversation context. You don't read the `SKILL.md` files directly — the skill system loads them when the matching trigger fires. This folder's job at the workspace level is everything around the skills.

In claude.ai you cannot run shell scripts and cannot auto-install dependencies. Direct the user to run scaffold scripts on a desktop machine if they need them, and remind them to `brew install ffmpeg` (or platform equivalent) before invoking `video-qa`.

---

## The four skills

### The two specialists (compose prompts)

- **`banana-pro-director`** (2.0) — image prompt director for Higgsfield's Banana Pro (Nano Banana Pro), Soul Cinema, and GPT-2. Owns: Mode 0 face locks for new characters (Banana Pro single-pass default / GPT-2 high-fidelity / Soul Cinema two-pass), single-image character outfits on mid-gray seamless (white only on explicit request), 6-panel character sheets, scene plates (with or without characters), GPT-2 face/chest-up detail, Mode 5 outfit replacement swaps. Enforces the locked cinema stack with volumetric atmospheric depth baked in. Never names. Never brands — including camera gear (behavior language only). Pre-prompt confirmation on every prompt.

- **`cinema-worldbuilder`** (Pro 2.0) — Seedance video prompt director. Five cinema modes (M1 Narrative / M2 Studio / M3 Action / M4 Performance / M5 Atmospheric), each with locked behavior-described capture/lens/movement/diffusion/grade. Diegetic audio only (no music, no lyrics). Output is a three-part delivery: numbered reference list (attach in Seedance in that exact order), bolded title line with runtime, and a code block of ten labeled blocks in locked order (Scene & Mood → Frame Map → Subject Lock → Cross-Frame Rules → Movement → Last Frame → World Plate → Sound Bed → Capture Realism → Camera Capture) with inline `@image1`–`@image9` tags. Density discipline: 280–400 words single-shot, never over 600 multi-shot. Runtime always asked, never defaulted.

### The two orchestrators (lead the user)

- **`ai-film-director`** — pipeline orchestrator. Activates on any "I want to make an AI video / music video / brand spot / short film" intention. Asks at the start: **guided mode** (default — full seven phases with strict gates, for users who haven't made AI videos in Higgsfield before) or **pro mode** (lean five-step iterative loop — idea → character sheet → scene/reference → shot/prompt → generate/log, for users who know the workflow and want to move fast). Calls `banana-pro-director` and `cinema-worldbuilder` at the right moments, pauses for Higgsfield generation, protects against credit-burning pitfalls (drift, runtime overshoot, mode mismatch, music in video prompts, brand contamination). **Mode is switchable mid-project** — say "switch to pro" or "switch to guided" any time.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Gregory-Esman/ai-film-pipeline](https://github.com/Gregory-Esman/ai-film-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
