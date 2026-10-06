---
trigger: always_on
description: This file is the project entry point for general AI agents.
---

# AGENTS.md

This file is the project entry point for general AI agents.

**You MUST read [`skills/ppt-master/SKILL.md`](skills/ppt-master/SKILL.md) before any PPT generation task or repo modification.** It owns global execution discipline and points to the route selector; after routing, the selected runtime authority owns its steps, gates, and commands.

**Repository execution anchor**: resolve the absolute repository root from this
file's supplied path and retain the absolute `skills/ppt-master` root before the
first command. Paths in this file are repository-relative notation only; invoke
them through those absolute roots, retain the absolute project path returned by
initialization, and never issue `cd skills/ppt-master` or `cd projects/...`.
When parsing machine-readable stdout, keep stderr separate and never place
`2>&1` upstream of a JSON or XML parser. Invoke each such command once per
concrete argument set; never encode its executable or flag list in scalar shell
strings, batch it through a shell loop, or add a downstream parser when the
command provides a compact view.

## Project Overview

PPT Master turns source material into natively editable DrawingML PPTX. Generate has two mutually exclusive runtimes: Default Strategist → Image_Generator → Executor, and self-contained Quick without separate strategy/confirmation. Beautify selects from explicit Quick intent; Image to PPTX always uses Quick.

**Route selection authority**: [`skills/ppt-master/workflows/routing.md`](skills/ppt-master/workflows/routing.md) owns the three top-level artifact routes: Generate PPTX, Create Template, and Edit Native PPTX. Child workflows, profiles, stages, and governance documents refine one selected route; they are not competing top-level routes.

- Topic-only or fact-insufficient inputs run [`topic-research`](skills/ppt-master/workflows/stages/topic-research.md) inside the selected Generate profile's source intake; its facts URLs are not auto-expanded. After normal image search fails, one relevant webpage may be fetched as a source package and only reviewed selections enter the runtime image pool.
- Default Generate prepares template candidates internally in Step 3, then confirms the communication contract and free-design/template choice together in Stage 1. Template content stays unread until that confirmation; selected roots are installed before template-aware Stage 2. Quick skips this interaction.
- Raw PPTX template plus new material/topic routes to [`edit-native-pptx`](skills/ppt-master/workflows/edit-native-pptx.md): a `pptx_to_svg.py --roundtrip` workspace where unchanged pages are referenced byte-for-byte and only planned pages are edited; it never enters Generate.
- Raw PPTX cannot be consumed as a Generate template workspace; run [`create-template`](skills/ppt-master/workflows/create-template.md) first and return with the generated workspace root as a Stage-1 candidate. Never add Master/Layout structure directly to an existing PPTX/SVG; generate new structured SVG pages from the workspace.
- Explicit quick/fast or skip-strategy generation uses [`quick-generate`](skills/ppt-master/workflows/profiles/quick-generate.md): prepare sources/resources as needed, decide without interaction, omit Strategist/confirmation/spec/lock, hand-author `svg_output/`, pass its lockless final checker, and export.
- Recorded, self-running, or video-directed Generate work conditionally loads [`video-design`](skills/ppt-master/references/video-design.md) inside the selected Default or explicit Quick runtime before page planning. It changes scene, script, and motion design—not the runtime/profile or artifact route.
- PPTX beautify is a strict 1:1 Generate [`profile`](skills/ppt-master/workflows/profiles/beautify-pptx.md), not a separate route. Explicit Quick intent uses the Quick runtime; otherwise it uses Default. Any split/merge/drop/reorder disables Beautify and returns to ordinary Generate in the selected runtime.
- Page-image reconstruction uses the Codex-supported, Quick-only [`image-to-pptx`](skills/ppt-master/workflows/profiles/image-to-pptx.md) profile. Normalize input page frames; one frame becomes one slide. Restore text natively, reconstruct low-resolution graphics without changing identity, and derive registered clean-base/scene layers. Padded-bbox-disjoint objects may share a generated plate and become independent crops. Never use a full-slide screenshot skin. Other hosts are unsupported.
- Finished PPTX notes / narration / timings / transitions with visible slides untouched also use [`edit-native-pptx`](skills/ppt-master/workflows/edit-native-pptx.md); export must report `rebuilt=0`.
- [`visual-review`](skills/ppt-master/workflows/stages/visual-review.md), [`customize-animations`](skills/ppt-master/workflows/stages/customize-animations.md), and [`generate-audio`](skills/ppt-master/workflows/stages/generate-audio.md) are supporting stages; their trigger rules remain explicit/conditional.

## Execution Requirements


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luonghaianh1208/2anh-studio](https://github.com/luonghaianh1208/2anh-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
