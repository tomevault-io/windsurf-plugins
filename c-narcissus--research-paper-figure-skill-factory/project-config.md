---
trigger: always_on
description: Use when the user wants a research-paper figure Skill Factory: build, patch, package, or use reusable specialized paper-figure-making skills from lawful literature/corpus evidence. Generated skills must use a specialized-skill-first workflow, full-feasible local PDF coverage where available, startup-plan-only first replies, target-paper candidate/final image isolation, mandatory image-embedded visual-structure explanation, mandatory non-target concept/modeling example display for abstract visual
---


# Research Paper Figure Skill Factory

This skill is a two-layer research-paper figure Skill Factory.

1. **Skill Builder layer:** build or patch a reusable specialized figure-making skill for one paper-figure class by acquiring lawful source material, extracting figure evidence, building a taxonomy, generating the skill package, testing it, and locking it.
2. **Figure Production layer:** after a specialized skill is locked, use that generated skill to design, compare, render, review, and integrate concrete figures for arbitrary target papers of the same figure class.

Version 2.0.5 adds a stricter visual-structure-as-image gate for generated specialized skills. When a generated skill explains or defines visual structure, layout skeleton, panel choreography, module topology, arrow grammar, candidate-board structure, second-round optimization geometry, or final content architecture in a text turn, it must show that structure with an embedded saved reference image or non-target concept/modeling example image. It must not substitute a prose-only or bullet-only visual-structure description. The existing hard gates remain: abstract visual decisions require inline reference/concept images, and after P6 selects the strongest first-round direction, P6b/P6b-IMAGE/P6c must run a paper-local best-practice optimization round before P7 final prompt construction. Target-paper candidate images, draft figures, final figures, and revisions still remain isolated in dedicated `IMAGE_ONLY` turns.

## Non-Negotiable Contract

### First Trigger

On first trigger, output only a startup plan. Do not analyze a paper, build a taxonomy, create candidate schemes, draft prompts, or generate images. The first reply is `STARTUP_PLAN_ONLY (TEXT_ONLY)`.

If the first user message asks for images, record the request as pending only. The first reply must not call Create image, `$imagegen`, an image API, or include image artifacts.

### Specialized-Skill-First Builder Rule

The normal route is:

`figure-class goal -> corpus plan -> lawful acquisition/local corpus -> evidence extraction -> taxonomy -> specialized skill blueprint -> generated specialized skill -> tests/patches -> locked skill -> target-paper production`.

Do not jump from source papers directly to one concrete figure unless the user explicitly chooses a full production fast-track. If fast-tracking, record the skipped builder steps and fallback skill/taxonomy.

### Full-Feasible Corpus Rule

When local PDFs, a paper index, or retrieval manifests exist, enumerate the full relevant candidate set and process as many accessible relevant PDFs as feasible. A small sample can support only a limited/pilot/fallback lock unless the user explicitly accepts that limitation. Representative rendered pages are audit aids only, not the corpus size.

### Mandatory Candidate-Image Bridge

Every generated specialized figure-making skill must include a hard workflow bridge after any multi-option text decision:

1. `TEXT_ONLY` candidate text turn: present 4-6 text candidates, normally 6.
2. `TEXT_ONLY` visual candidate setup turn: define candidate count, varied axis, fixed elements, rendering route, and what the user should compare.
3. `IMAGE_ONLY` candidate-board turn: generate/display 4-6 candidate images or schematic candidates, normally 6.
4. `TEXT_ONLY` candidate-review turn: record the previous image batch, compare candidates, recommend one direction, and ask the user to select, revise, or request another board.

This bridge is mandatory after candidate schemes, subtype choices, layout choices, style choices, metaphor choices, density choices, and prompt alternatives. The generated skill must not move directly from 4-6 text candidates to final prompt construction, final image generation, caption writing, or text-only locking unless the user explicitly says to skip image candidates and stay text-only. If skipped, record `visual_candidate_board_skipped_by_user: true`.

Generated skill lock/test must fail if:

- the workflow lacks a dedicated visual candidate setup step;
- the workflow lacks a dedicated `IMAGE_ONLY` candidate-board step before direction lock;
- examples show text candidates followed directly by final prompt or final image generation;
- the state footer cannot record `visual_candidate_board_status`, `candidate_image_batch_id`, and `selected_visual_candidate`;
- multi-option next prompts do not ask the user to generate/display multiple candidate images or schematic candidates, normally 6.

### Target-Paper Image Isolation And Required Inline Reference Display

Every response must distinguish target-paper figure production from explanatory reference display:

- `TEXT_ONLY`: planning, intake, diagnosis, candidate text, candidate-board setup, prompt writing, critique, status, next prompts, and inline display of allowed reference images.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [c-narcissus/research-paper-figure-skill-factory](https://github.com/c-narcissus/research-paper-figure-skill-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
