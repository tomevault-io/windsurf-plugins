---
trigger: always_on
description: This is a standalone Toolcraft template app generated from the base starter.
---

# Toolcraft App Template Assembly Guide

This is a standalone Toolcraft template app generated from the base starter.

## Required Preflight

Treat this `AGENTS.md` as the active project contract. Before planning or editing app code, runtime code, controls, canvas, panels, renderer, timeline, layers, export, or tests, you must read:

1. `docs/toolcraft/workflow.md`

Then follow `workflow.md` to choose the required contract docs and verification tier. Do not edit implementation files until this preflight is complete.

## Quick Entry Contract

1. Build through `defineToolcraft` and `ToolcraftApp`.
2. Keep app state in Toolcraft runtime schema and commands.
3. Keep product output in `canvasContent`; never render app UI there. If upload/import is part of the source-material flow, do not invent canvas placeholder artwork, CTA copy, helper text, fake sample output, or preset source designs before real content exists.
4. Use built-in Toolcraft controls before custom controls.
5. Do not hand-compose runtime surfaces or render built-in control components directly in app code; use `ToolcraftApp`, schema controls, `canvasContent`, `controlRenderers`, `onPanelAction`, and runtime commands.
6. Before writing controls, make and export `starterControlSectionInventory`: each product controls section declares its title, product entity or workflow stage, targets, and grouping reason. Group by product meaning, not UI component type.
7. Keep control `label` short but semantically sufficient with the nearest visible section/group context, and put product-specific behavior help in schema `description`; runtime renders the label help tooltip only when that description adds meaning beyond the label.
8. Enable layers and timeline only when product behavior requires them, then test the real UI. Product animation loops are seamless forward-only by default: first and last frames stitch, direction does not reverse, and mirror/yoyo/ping-pong behavior requires explicit user intent.
9. Animated preview renderers suspend or coalesce non-essential animation work during canvas drag, pan, pinch, zoom, and radar/center interactions, then resume without changing user playback state.
10. If a Figma URL is provided, inspect the Figma file through MCP and rebuild from its structure; never implement from a screenshot or by eye.
11. If a video, GIF, screen recording, contact sheet, or extracted-frame sequence is provided as a reference, write a Video Reference Study before implementation: storyboard frames, frame-to-frame transition analysis, behavior decomposition, and acceptance mapping. Do not implement video references from a single screenshot or high-level summary.
12. Choose an explicit persistence policy; use schema `persistence` for user-edited app settings that should survive reload, and test real reload restoration when localStorage is enabled.
13. Generated apps follow the mandatory runtime Setup, canvas sizing, render scale, Timeline switch, Background, Image Export, Video Export, and sticky action rules in `docs/toolcraft/core/setup-export.md`. Do not duplicate or reinterpret those controls in app-authored sections.
14. Media uploads, image/file mode, source images, multi-upload sorting, default assets, and image transform actions follow `docs/toolcraft/core/media-upload.md`.
15. Keep `docs/toolcraft/agent-worklog.md` current with a decision trail, product decisions, explicit reference inputs, evidence, verification, and risks. Reference-runtime-clone apps also declare `referenceStudy` plus `referenceFeatureInventory` so every inspected reference feature has feature-level behavior evidence and maps to Toolcraft implementation and acceptance coverage.
16. Prove every visible entity through acceptance, browser, and performance coverage.
17. Performance workload, render scale, live slider responsiveness, GPU evaluation, and optimization evidence rules live in `docs/toolcraft/core/performance.md` plus `docs/toolcraft/performance.md`.
18. Custom renderer apps declare a Render Pipeline Inventory in typed `rendererPipeline`: render passes, cache keys, execution location, preview/export quality, and interaction invalidation.
19. Classify every implementation pass with a verification tier before editing. Use targeted checks for incremental edits and the full final gate only for final delivery, exports, or architecture/runtime/template changes.

## Starter Baseline

The generated folder starts as a neutral Toolcraft shell: canvas upload plus toolbar. It intentionally does not include demo controls, prompt fields, layers, or timeline. Do not treat test fixtures or documentation examples as product requirements. Add controls, timeline, layers, sticky actions, persistence, and custom renderers only after the requested product or reference app requires them.

When the folder becomes a real product, update `src/app/app-acceptance.ts` from `appProductReadiness.mode: "starter"` to `mode: "product"` and fill `productName`, `productSummary`, and `requestedBehavior`. Renamed product folders are not allowed to keep neutral starter readiness.

## License


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pixel-point/toolcraft](https://github.com/pixel-point/toolcraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
