---
trigger: always_on
description: This file should always be used as the entrypoint for agents working in this repository. Keep it generic and concise.
---

# AGENTS.md - Agent Guidance

This file should always be used as the entrypoint for agents working in this repository. Keep it generic and concise.
Project-specific standards live under `.agents/standards/` and task-specific guidance lives in the relevant skill files
under `.agents/skills/`.

## Before Editing

- Read `.agents/standards/openhound.md` before making OpenHound collector changes.
- Read `.agents/standards/workflow.md` before developing a new collector or making broad collector changes.
- Read `ARCHITECTURE.md` before touching any cross-cutting collector subsystem (the per-host phased
  pipeline, recursive discovery / target allow-list, the Windows authentication stacks under `clients/`,
  the logging/diagnostics layer, the Windows-specific fixes, or the preproc/convert design). It explains
  how and why this extension diverges from a stock OpenHound (REST-API-only) collector. **Update the
  relevant section of `ARCHITECTURE.md` in the same change** whenever you alter one of those subsystems,
  and fix any `file:line` references your change invalidates. Add a new section if you introduce a new
  category of divergence.
- **`openhound-collector-common` is a separate published package — treat it exactly like `openhound`
  core.** The Windows auth stacks, the per-target logging layer, the push→pull streaming bridge
  (`StreamBridge`), and the DNS resolver *live there*, shared with the MSSQL collector; this repo's
  `clients/*`, `log_context.py`, and `phased_pipeline/streams.py` are thin adapters/re-exports over it
  (see `ARCHITECTURE.md` → "Where this code lives"). It is declared as a capped version range in
  `[project.dependencies]`; `[tool.uv.sources]` optionally redirects it to a sibling checkout at
  `../openhound-collector-common` for local work, and that redirect never reaches the published wheel.
  Trace into the library to understand behaviour, but **do not treat editing it as part of a change
  here** — it affects both collectors and it releases on its own tag. If a task seems to need a
  shared-library change, stop and say so.
- Load the `openhound` skill from `.agents/skills/openhound/` for task-specific workflows.

## Task Skill

Use `openhound` for all OpenHound collector work. The skill routes tasks to action-specific references.

| Task                                                                                   | Skill       | Reference |
|----------------------------------------------------------------------------------------|-------------|---|
| Plan a new collector from target service requirements or API docs                      | `openhound` | `.agents/skills/openhound/references/plan-collector.md` |
| Add or modify a collected asset/model                                                  | `openhound` | `.agents/skills/openhound/references/add-asset.md` |
| Implement API collection resources, transformers, auth and DLT source wiring           | `openhound` | `.agents/skills/openhound/references/source-collection.md` |
| Define base graph node/edge dataclasses and ID generation behavior                     | `openhound` | `.agents/skills/openhound/references/graph-schema.md` |
| Add DuckDB transforms or lookup methods                                                | `openhound` | `.agents/skills/openhound/references/preproc-lookup.md` |
| Wire phase registration (collect, preproc, convert), metadata, or package entry points | `openhound` | `.agents/skills/openhound/references/register-extension.md` |
| Validate a collector before finishing                                                  | `openhound` | `.agents/skills/openhound/references/validate-extension.md` |

## General Rules

Behavioral guidelines. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:

- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:

- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:

- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SpecterOps/ConfigManBearPig](https://github.com/SpecterOps/ConfigManBearPig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
