---
trigger: always_on
description: >
---


# PPT as Code

> Plan and build HTML-based presentations with a creator-first staged workflow.

**Core Pipeline**: `Ingest -> Normalize Source Material When Needed -> Derive Source Scenes When Needed -> Detect Input Mode -> If DSL: Parse deck.md -> Compile deck_source.json -> Route Mode -> Load References -> Diagnose Gaps -> Produce Artifacts -> Confirm Key Decisions -> Plan Visualizations -> Plan Images -> Run Pre-HTML QA -> Deliver HTML -> Optional Workbench Sync -> Optional PPTX Export`

---

## Mandatory Rules

> [!CAUTION]
> ### Serial Execution & Gate Discipline
>
> This workflow is a strict serial pipeline.
>
> 1. **SERIAL EXECUTION**: Steps must execute in order; output of each step becomes input for the next.
> 2. **BLOCKING = HARD STOP**: A `BLOCKING` step requires explicit user confirmation before continuing.
> 3. **NO SPECULATIVE EXECUTION**: Do not pre-build future artifacts before the current gate is cleared.
> 4. **KEEP THE ROUTE PROPORTIONAL**: Do not escalate a lightweight deck into a heavier stack unless the request clearly needs it.
> 5. **DEFAULT TO SHIPPING**: Deliver usable artifacts, prompts, or HTML handoff routes, not theory essays.
> 6. **FINAL ARTIFACT MUST RUN**: If the user wants a direct handoff, prefer a self-contained file or a locally runnable folder over a CDN-only prototype.

> [!IMPORTANT]
> ### Strict Execution Semantics For `basic` And `advanced`
>
> - Treat `basic` and `advanced` as strict finite-state workflows, not loose checklists.
> - Advance only one stage at a time. Do not silently bundle multiple blocking stages into one jump.
> - Silence, implied urgency, or generic requests such as "continue" do not count as permission to bypass required confirmations.
> - Only bypass a blocking step when the user explicitly names that override in a clear instruction, for example:
>   - `skip breakdown confirmation`
>   - `do not ask for step-by-step confirmation`
>   - `run the rest end-to-end without checkpoints`
> - If the user gives a partial override, skip only the named checkpoint and keep the rest of the workflow strict.
> - If the user does not clearly opt out, stay in the default strict step-by-step mode.

> [!IMPORTANT]
> ### Creator-First Delivery
>
> - Default delivery is **staged artifacts + HTML route or prompt pack + image plan**, not a long code dump.
> - When the user is a creator or wants to move fast, prefer staged prompts and intermediate artifacts over large code blocks by default.
> - Use full implementation only when the user explicitly asks for code or the workflow has reached the final HTML stage.

> [!IMPORTANT]
> ### PPTX Export Handoff
>
> - `ppt-as-code` remains an **HTML-first** presentation skill.
> - PPTX export is an optional **final-delivery post-process**, not a replacement for the HTML workflow.
> - Default export target is `html`; allow `pptx` or `both` when the user explicitly wants PowerPoint delivery.
> - When the target includes PPTX, finish the static HTML pass first, then produce `deck_manifest.json`, then hand off to `pptx-export-for-ppt-as-code`.
> - `deck_manifest.json` is the export bridge and source of truth for PPTX delivery; do not rely on ad-hoc DOM scraping as the primary route.
> - PPTX export is static-only: motion is downgraded to a static state, simple pages should stay editable, and complex pages may fall back to full-slide raster images.

> [!IMPORTANT]
> ### Slidev-Inspired DSL Input
>
> - `ppt-as-code` may accept an optional `deck.md` draft as a Slidev-inspired authoring input.
> - This is a **draft input layer**, not a compatibility promise for real Slidev projects or syntax.
> - When `deck.md` is present, parse it into `deck_source.json` before entering the normal `quick`, `basic`, or `advanced` workflow.
> - `deck.md` can speed up ideation, but it must not bypass `basic` or `advanced` confirmation gates.
> - `deck_source.json` is the normalized internal representation for DSL input; it is not the final delivery artifact.

> [!IMPORTANT]
> ### Source Normalization Rules
>
> - If the user starts from external source material rather than a slide outline, normalize that material before deck breakdown or script work.
> - When these adapters are available in the environment, use them proportionally:
>   - PDF -> `pdf_to_md.py`
>   - DOCX / EPUB / HTML / LaTeX -> `doc_to_md.py`
>   - ordinary web pages -> `web_to_md.py`
>   - high-friction pages such as WeChat or anti-bot pages -> `web_to_md.cjs`
> - The normalized markdown becomes the working source for deck planning; it is not the final deck.
> - If no adapter is available, do not block the workflow. Summarize the source manually and continue with an explicit note about the fallback.

> [!IMPORTANT]
> ### Source-To-Scenes Rules
>
> - If the starting material is a long article, PDF, document, or normalized web source, derive a preliminary scene map before writing the confirmed breakdown.
> - The source-to-scenes pass should cluster the material into likely slide groups, not final slides.
> - Use it to identify likely page sequence, candidate scene roles, strong quotes, strong stats, and image-worthy sections.
> - Treat `source_scene_map.md` as a planning accelerator, not as a replacement for the confirmed breakdown.

> [!IMPORTANT]
> ### Persistence Strategy
>
> - Default to **conversation-first artifacts**.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Russell-cell/PPT-as-code](https://github.com/Russell-cell/PPT-as-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
