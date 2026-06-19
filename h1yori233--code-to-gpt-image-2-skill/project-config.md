---
trigger: always_on
description: Generate a structured GPT Image 2 prompt JSON from any codebase. Use when Codex needs to read a repository, source file, app, CLI, library, or architecture and produce a prompt.json for a technical infographic, architecture diagram, pipeline explainer, module map, data-flow diagram, or educational codebase visualization.
---


# Codebase To GPT Image 2 Prompt

Turn a codebase into a structured GPT Image 2 prompt JSON for a high-quality technical infographic. The goal is not to document every file or wrapper; the goal is to extract the most important mechanism in the code and encode it as controllable image-generation JSON.

## Workflow

1. **Resolve scope**
   - Use the provided folder, file, GitHub repo, or current working directory.
   - If the user names a specific entry point, treat it as the anchor.
   - If no entry point is named, infer it from README, package/project config, CLI files, routes, app bootstrap files, or common framework conventions.

2. **Read strategically**
   - Start with README or docs, dependency/config files, and primary entry points.
   - Treat README/docs as orientation only: scope, terminology, supported modes, documented defaults, and user-facing examples. Unless the user explicitly asks for a README-only diagram, verify P0 mechanics from implementation files.
   - Use `rg` / `rg --files` to locate important modules, commands, routes, classes, and public APIs.
   - Prefer targeted batches of 2-5 files. Do not bulk-load a large file set and then summarize everything at once.
   - Read past wrapper code into the downstream modules that do the real work.
   - Classify files and functions as **core mechanism**, **orchestration**, **boundary adapter**, or **runtime glue**.
   - Read enough core code to explain what the system transforms, which components do the transformation, and why the design is interesting.
   - After finding the likely P0 mechanism, perform a second depth pass on the implementation files behind it. Do not stop at the entry-point call site.
   - For tensor, media, ML, compiler, database, or protocol code, derive a concrete worked example when the user gives a scenario or the repo has a clear default. Include formulas and numeric values, not only symbolic shapes.
   - Do not exhaustively analyze unrelated implementation details unless they affect the visual story.

3. **Build an internal evidence ledger**
   - Maintain the ledger internally during the task. Do not save `evidence.json`, do not make the ledger the final answer, and do not stop after code analysis.
   - First pass: build `source_map` from docs, configs, entry points, and quick searches. Record P0/P1/P2 candidates and the next implementation files to read.
   - Second pass: read only the selected P0 implementation files in small batches. After each batch, update `mechanism_cards` with claim, source, priority, `importance_score`, `why_it_matters`, `visual_budget`, `compression_strategy`, visual role, and unresolved questions.
   - Third pass: for numeric systems, update `shape_ledger` with scenario, formula, concrete value, source, and `attach_to_panel`. Keep only numbers that explain representation changes, compute scale, state flow, cache/chunk behavior, or output shape; not every shape deserves visible space.
   - Track data lineage in the ledger. When a later tensor, state, or object uses an earlier value, record the producing panel and consuming panel.
   - Before writing the final storyboard, perform a `budget_pass`: move canvas from low-value setup, input branching, repeated formulas, and easy arrows toward core loops, model architecture, state updates, and decode/materialization.
   - Stop exploring when the ledger is sufficient to draw the core mechanism. Then generate the prompt JSON from the ledger.

4. **Rank information by visual importance**
   - **P0 - Core mechanism**: domain concepts, algorithms, model architecture, data transformations, state machines, request/data flow, core loops, storage semantics, rendering logic, or protocol behavior.
   - **P1 - Supporting architecture**: orchestration branches, component boundaries, adapters, external services, persistence, output creation, extension points, and failure handling.
   - **P2 - Runtime glue**: CLI argument parsing, environment setup, logging setup, process rank setup, config defaults, file naming, and generic save/load code.
   - Build the infographic from P0 and P1. P2 should usually be omitted; if it appears, it may only be a tiny caption, footer, or compact callout.
   - Never spend the first 2-3 main panels on CLI/runtime setup unless the codebase is itself a CLI framework or runtime tool.
   - Allocate more space to mechanisms with hidden complexity, state mutation, repeated execution, performance/quality impact, or representation changes.
   - Compress details that are simple input branching, routing, default choices, repeated formulas, or one-step combinations that an arrow can explain.
   - For each panel, ask: if this panel were deleted, would the viewer still understand the core mechanism? If yes, shrink it, merge it, or turn it into an inline callout.
   - For ML/video/diffusion systems, conditioning, input setup, and runtime setup together should usually use only 15-25% of the canvas. Denoising/model architecture/decode should usually use at least 70%.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [H1yori233/code-to-gpt-image-2-skill](https://github.com/H1yori233/code-to-gpt-image-2-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
