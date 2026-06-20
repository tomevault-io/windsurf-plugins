---
trigger: always_on
description: Windows-first Visio diagram reconstruction workflow for flowcharts, architecture diagrams, and paper-style module figures. Reuses ppt-master style analysis and composition discipline on the front half, but outputs editable Visio .vsdx plus exported .svg and .png through a scene.json to Visio pipeline. Use when the user wants a diagram recreated as editable Visio shapes instead of a pasted screenshot or PPT-only result.
---


# Visiomaster

## Overview

`visiomaster` is a standalone skill for rebuilding diagram images into editable Visio deliverables.

It is optimized for:
- flowcharts
- product or system architecture diagrams
- paper-style module/framework figures
- box-arrow process diagrams that should remain editable

It is not the right tool for:
- posters
- UI screenshots
- decorative layouts
- image-heavy slides where the main value is visual styling rather than structured diagram semantics

## Core Positioning

Use `ppt-master` ideas on the front half:
- source collection
- style extraction
- layout discipline
- image understanding
- visual polishing standards

Do **not** reuse `ppt-master`'s raw `SVG -> PPTX` output path for Visio.

For Visio, the stable path is:

`image -> scene.json -> validate -> Visio COM render -> .vsdx/.svg/.png`

The key rule is simple:
- main structure should be redrawn as editable nodes, labels, and connectors
- small thumbnails or texture snippets may remain raster only when redrawing them is not worth the loss in speed
- never solve a reconstruction request by pasting the whole original image unless the user explicitly asks for贴图

## Environment

This skill is Windows-first and expects:
- local Microsoft Visio desktop installed
- Python with `pywin32`

Use the active thread default Python interpreter. If the user or project provides a specific Python path, use that; otherwise use:

```powershell
python
```

## Workflow

### 1. Confirm scope

First classify the source request:
- editable flowchart recreation
- architecture/module diagram recreation
- paper figure cleanup/redraw
- image-assisted redraw with a few raster sub-assets allowed

If the diagram is mostly boxes, arrows, labels, and containers, stay in `visiomaster`.

For image-based exact replicas, always secure a local source image file before claiming strict review readiness:
- If the user provides a filesystem path, use that path as `metadata.source_image` and all review `--original` inputs.
- If the user uploads or attaches an image without a local path, save/stage that image into the reconstruction workspace as `source/original.<ext>` before rendering the final deliverable. Do not treat a chat attachment alone as an acceptable strict-review source.
- If the current client does not expose an attached image as readable bytes and the image cannot be staged locally, clearly mark the work as a draft/manual preview and ask for a source file path before strict review or final exact delivery.
- Every exact replica must keep the staged/local source path stable across review rounds. Do not use screenshots of the replica, pair images, OCR output, or memory of the attachment as a substitute for the original source image.

When you have a source file path, stage it with the helper script so review manifests can use one stable canonical image:

```powershell
python ${SKILL_DIR}\scripts\stage_source_image.py `
  --input <source.png> `
  --workspace <reconstruction_workspace> `
  --id <figure_id>
```

This writes `source/original.<ext>` and `source/source_manifest.json` with SHA-256 hashes. Use the staged image path as `metadata.source_image` and as `make_review_assets.py --original`.

For wide or dense figures, do not start by authoring the whole page in one pass. If the source has many modules, many arrows, tiny labels, or a very wide canvas, first create a region plan:
- visible regions become `group_container`
- invisible logical work areas become `audit_region`
- every meaningful node gets `container_id`
- each region should usually stay under 12-18 visible nodes before whole-page assembly
- shared typography and arrow styles must be fixed before region scenes are merged

For exact replicas, do not write `scene.json` until you have a source-faithful visual inventory for that specific image. This inventory is produced by visual LLM inspection of the source image, not by OCR, filename clues, warning logs, prior scenes, or a batch scene generator. It must preserve visible source language and notation exactly; do not translate Chinese labels to English, normalize formulas, invent unreadable text, or replace source modules with generic neural-network blocks unless the source visibly shows them.

Record the inventory in `metadata.source_visual_inventory` before the first render. At minimum it should include:
- `analysis_basis`: a short note such as `visual_llm_source_image`.
- `language_profile` and `do_not_translate`.
- `unknown_text_policy`: use `mark_unreadable_do_not_invent` for unclear tiny text.
- per-region `source_bbox_px`, required visible labels/formulas, component motifs, edge motifs, port/boundary/dashed-frame notes, uncertainty notes, and text layout facts.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Rss3208/Visiomaster](https://github.com/Rss3208/Visiomaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
