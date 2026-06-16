---
trigger: always_on
description: Use Paopao to turn PDFs, reports, papers, and reference images into editable consulting-style PPTX decks. Trigger when the user asks to make, generate, create, or package a PPT/PPTX/slides/deck from source documents, especially when they want editable output rather than image slides.
---


# Paopao PPT

paopao creates editable PowerPoint decks from source documents.

## Hard Rules

- Do not freehand the workflow from memory. At the start of every Paopao task and after each major artifact is created, run `paopao_run.py run-task --task-dir output/<task-name>` and follow the blocked `next_action`. If `run-task` is blocked, do not skip ahead to later stages.
- Final output must be `.pptx`.
- Final PPTX must be editable. Never use a whole-slide screenshot, Image2, PNG, JPG, or PDF as the slide background.
- Image2 visual references are required before any reconstruction source. Do not skip Image2, do not substitute old drafts, and do not write HTML or PPTX from memory.
- Image2 visual references must be landscape 16:9 widescreen, target 1920x1080. Portrait, square, 4:3, and 3:2 references are invalid and must be regenerated.
- Each `final_prompt_XX.md` must be filled from one concrete prompt-library file in `<plugin-root>/prompts/` and declare `PROMPT_TEMPLATE: <file>.md` plus the matching `LAYOUT_NAME`. Handwritten layout prose that merely resembles the style is invalid.
- Prompt templates must be selected through `analysis/slide_story.json` and `plan-prompts` before writing final prompts. Do not manually browse the prompt library and choose by habit; final prompts must match `analysis/prompt_selection_plan.json`.
- Each Image2 reference must be generated from the corresponding template-backed `final_prompt_XX.md` plus the Paopao compact consulting style rules below. Generic image prompts or decorative mockups are invalid.
- Preserve prompt-library layout diversity. The visual style rules control palette, typography, linework, borders, icons, background, and material simplicity; they must not force every slide into one repeated composition.
- The only valid input to the image generation tool is the `prompt_text` field in `output/<task-name>/image2/generation_request_XX.json`, produced by `prepare-image2-prompts`; that field must exactly equal `output/<task-name>/image2/image2_prompt_XX.md`. Do not handwrite, summarize, compress, translate, or improvise image-generation prompts in chat/tool calls.
- After saving selected Image2 references, register each one with `register-image2-reference` and the exact `image2_prompt_XX.md` sha. Do not rerun `prepare-image2-prompts` as a substitute for registration. `check --stage image2`, `check --stage html`, and `render` must fail if locked Image2 provenance is missing, stale, or unregistered.
- After Image2 is generated, treat each generated image as an unfamiliar external reference. Open it again, observe it, record `qa/image2_style_review.json`, then reconstruct only from image-derived measurement.
- After Image2 references are generated and internally reviewed, pause and show the selected page previews to the user. Do not write specs, HTML, or render PPTX until the user explicitly approves the previews and `record-image2-user-review` has recorded that approval. If the user requests changes, regenerate the affected references first, then ask again.
- After user approval and before writing specs, run `forget-after-image2` to lock the post-image memory boundary. From that point on, do not consult or reuse `final_prompt_XX.md`, `image2_prompt_XX.md`, prompt templates, analysis notes, or remembered intent for visual reconstruction. `record-image2-observation` also auto-creates this boundary if it is missing, but the preferred workflow is to lock it explicitly immediately after approval.
- After the memory boundary is locked, run `record-image2-observation` once per slide from fresh visual inspection of the selected image. Specs, visual contracts, HTML, and direct PPTX render plans must bind to that observation record by `observation_id`; do not let them derive directly from prompts, analysis, or memory.
- After recording the fresh observation, run `extract-image2-contract` for each slide before writing or editing the visual contract. This command creates the initial region inventory from the selected image pixels. Manual edits may correct OCR/text labels or refine regions, but the starting geometry must come from this image-extraction step, not from prompt memory.
- After user approval, treat each selected Image2 reference as if it were a brand-new user-supplied screenshot with no production history. Discard prompt, prompt-template, analysis, and layout-library memory for visual reconstruction. The spec, visual contract, and HTML must be authored from fresh observation of the actual image only, not from `final_prompt_XX.md`, `image2_prompt_XX.md`, `analysis_report.md`, or remembered slide intent.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kakoutang/paopao](https://github.com/Kakoutang/paopao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
