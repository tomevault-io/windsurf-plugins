---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **prompt-engineering toolkit** (no application code) that turns reference images into on-style 2D mobile-game assets. The deliverable is `studio_primer.md` — a self-contained mega-prompt the user pastes into a vision-capable LLM chat, which then answers short commands (`STYLE`, `UPDATE:`, `ASSET:`, `CHARACTER:`, `BACKGROUND:`, `OBJECT:`, `CHECK`, `REGEN`, `TWEAK`) to produce a `style_guide.yaml` and, from it, image prompts. The primer only ever outputs **text** — it never invokes an image tool even when the host chat has one.

**Draw-fresh only (deliberate design decision, 07/2026):** every asset is generated FRESH from the style_guide (eyedropper-verified hex pinned in the prompt) — that is the path generative models are good at. The former EXTRACT / UPSCALE / `STYLE: design system` board paths (re-rendering existing pixels) were removed after real-world testing showed them producing mushy, broken output on gpt-image; do not re-add "keep the source pixels" paths. The one image-edit path kept is the CHARACTER pose variation (identity lock). **The STYLE ref is attached only to `STYLE`** — later commands inherit the style_guide from context, and their attachment slot belongs to that command's own ref (TARGET layout / CHARACTER / the generated result for CHECK); don't reintroduce "re-attach the STYLE ref when generating" wording.

**Generator-neutral by design**: the toolkit stops at the prompt. It never generates images and must not be coupled to any specific image generator (gpt-image / Gemini "Nano Banana" / Midjourney are the user's choice). Scope is 2D assets in four classes: **UI/UX** (screens, icons, buttons, panels), **backgrounds**, **characters** (static art + pose variations via a CHARACTER ref; animation sheets are out of scope), and **objects/props**. Repo content stays in English (prompts target English-language generators).

## The one rule that matters: the primer is GENERATED

`studio_primer.md` is bundled from two source directories — never hand-edit it independently of them:

- `schema/` — field + enum definitions. `style_guide.schema.yaml` is canonical; `asset_spec.schema.yaml` / `layout_spec.schema.yaml` describe the optional structured UI asset/layout inputs; `background_spec.schema.yaml` / `character_spec.schema.yaml` / `object_spec.schema.yaml` the optional structured inputs for the three non-UI classes.
- `style_tokens/` — the style dictionary mapping each enum to an English prompt phrase (`materials`, `render_shape`, `light_color`, `layout_negative`, `character_environment`).

Change flow: edit the source file first → rebuild the affected primer section → verify against the **BUILD MANIFEST** in the primer's header comment (it lists every source section that must be present; grep each item). The schemas are documentation-style pseudo-YAML (fields + enums, not JSON Schema); there is no build/validator script — the repo intentionally stays code-free.

## Architecture (pipeline inside the primer)

STYLE ref image(s) → **§3 ANALYZER** → `style_guide.yaml` (enums from §1 only, plus per-dimension `confidence`; `<0.75` = user must review; hex values are estimates the user fixes via eyedropper + `UPDATE:`; the `character`/`environment` blocks are OPTIONAL — filled only when the refs actually contain characters/scenes, never guessed) → **§4 SYNTHESIZER** (the four generate commands map 1:1 to its four prompt branches — `ASSET:`=UI, `CHARACTER:`, `BACKGROUND:`, `OBJECT:`; each branch translates every enum through the §2 dictionary, in its fixed prompt order ending with "Avoid:") → one natural-language prompt → user generates the image elsewhere → optional **§5 CHECKER** compares the generated image back against the style guide — extra-strict on palette hex (per-role expected vs observed) and UI edge sharpness (blur/mush/soft text = off) — and emits per-fix `TWEAK:` lines plus **ONE consolidated `TWEAK:`** the user copies to fix everything in one go.

Character **pose variation** is a sub-branch of §4: `CHARACTER:` + an attached CHARACTER ref (the already-generated character; or `character_spec.character_ref`) → an image-EDIT prompt that opens with the §2 identity-lock phrase (face/outfit/colors frozen, only pose/expression change). It is the ONE image-edit path in the toolkit — it needs an image-*editing* generator (the §4 IMAGE-EDIT PATH NOTE covers this) but still translates lighting/outline through §2.

Two consistency sub-modes mirror each other (both stay inside their parent command — no new command, so §0/README stay 1:1): the **character sheet** (`character_spec.sheet`) draws one character's expressions/turnaround on a single sheet; the **UI-KIT sheet** (`asset_spec.kit` / `type: ui_kit`, or "UI kit" in text) is the `ASSET:` analogue — the whole UI component set (button/toggle/slider/checkbox/progress/panel/icons/text) drawn on ONE canvas in one pass so widgets can't drift apart. Both lean on the §2 word **consistent** as the anti-drift anchor. UI widget/text style itself lives in the OPTIONAL `typography` / `controls` style_guide blocks.

Distinctions that must not be blurred:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tnbao91/ai_asset](https://github.com/tnbao91/ai_asset) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-05 -->
