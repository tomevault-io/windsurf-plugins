---
trigger: always_on
description: Guidance for AI/code agents working in this repository. This file is the
---

# AGENTS.md

Guidance for AI/code agents working in this repository. This file is the
current project map after multiple RAW workflow, UI, runtime, planning, and
verification refactors. Keep changes aligned with this app, not generic Vite or
image-editor assumptions.

## Non-Negotiables

- Use `pnpm` only.
- Do not read, write, or commit `.env` files, secrets, credentials, or private
  tokens.
- Do not edit generated files such as `src/generated-routes.ts`. Change routes
  by adding or renaming files under `src/pages/`.
- Use the `~/` alias for imports from `src`.
- Stay inside the shared app runtime patterns: do not recreate the QueryClient,
  Jotai store, router plumbing, or provider stack outside the existing
  providers.
- Do not use `window.location` or other ad hoc navigation paths when existing
  router utilities cover the case.
- For animation, use `m` from `motion/react` inside the existing `LazyMotion`
  setup in `src/providers/root-providers.tsx`. Prefer presets in
  `src/lib/spring`.
- Do not describe or reintroduce the RAW runtime as `libraw-wasm`. The current
  runtime boundary is `@lumaforge/luma-raw-runtime`.
- Do not add catalogs, batch workflows, accounts, cloud upload requirements,
  local daemons, native helper apps, or broad desktop-editor panels unless the
  user explicitly asks for that product shift.
- Keep commits prompt, focused, and minimal. Do not add AI co-authorship
  metadata.

## Product Boundary

- LumaForge is a browser-local RAW photo lab for a narrow workflow:
  `single RAW file -> preview -> look or LUT -> compare -> JPEG export`.
- Preview may optimize for responsiveness through embedded, quick, bounded HQ,
  WebGL, or CPU-degraded stages.
- Export is the authoritative full-resolution path. If the runtime cannot prove
  the declared pipeline can be reproduced, fail closed instead of exporting a
  degraded or preview-only result.
- HQ preview export is a fallback or compromise, not the primary promise.
- Color and LUT work is contract work. Preserve input gamut, transfer/log curve,
  LUT intent, scene-referred working assumptions, and output handling.

## Current Architecture

- `src/pages/(main)/raw.tsx` is the `/raw` route entry. Route files drive
  `src/generated-routes.ts`; never edit the generated file directly.
- `src/providers/root-providers.tsx` owns `LazyMotion`, React Query, Jotai,
  i18n, error boundary, router stability, settings sync, context menu, and
  toasts. Preserve provider order unless a concrete bug requires changing it.
- `src/modules/raw-processor/RawProcessorView.tsx` is a thin composition layer.
  Keep orchestration in hooks/controllers and domain logic in services.
- `src/modules/raw-processor/hooks/useRawProcessorViewController.ts` bridges
  route/runtime state, hidden file pickers, runtime readiness, reset
  confirmation, CPU preview state, and workflow actions for the view.
- `src/modules/raw-processor/hooks/useRawWorkflow.ts` and
  `hooks/stages/*` are the workflow state machine boundary. Stage hooks are
  grouped by `ingest`, `preview`, `look`, `compare`, and `export`.
- `src/modules/raw-processor/services/*` contains scriptable domain behavior:
  `ingest`, `preview`, `look`, `compare`, and `export`. Prefer adding tested
  logic here before growing React components.
- `src/modules/raw-processor/model/*` defines session/workflow/export result
  shapes. Keep model changes small and contract-like.
- `src/modules/raw-processor/state/*` contains Jotai atoms for workflow and tool
  state. Prefer these over introducing a second state model.
- `src/modules/raw-processor/components/RawWorkflowToolProvider.tsx` is the
  context bridge from workflow state/actions to tool surfaces.
- `src/modules/raw-processor/components/RawToolSurface.tsx` switches desktop vs
  mobile surfaces by viewport. Respect that split.
- `src/modules/raw-processor/components/tools/*` is the desktop command rail:
  Adjust, Tone, Color, Compare, Export, File Facts, Histogram, LUT contract, and
  shared tool chrome.
- `src/modules/raw-processor/components/mobile/*` is the mobile photo-first
  shell: persistent topbar, bottom mode dock, mobile LUT browser, mobile export,
  compare panel, Adjust list panels, and scrub HUD.
- `src/modules/raw-processor/raw-lab.css`,
  `raw-lab.surface.css`, and `raw-lab.effects.css` hold `/raw` surface and
  effect CSS that cannot reasonably live as Tailwind utilities.
- `src/lib/gl` is the WebGL2 interactive preview renderer.
- `src/lib/preview` is the CPU/degraded preview worker path and capability
  helpers.
- `src/lib/export` is the worker-driven full-resolution export path.
- `src/lib/raw` adapts the app to `@lumaforge/luma-raw-runtime`.
- `src/lib/runtime` owns capability and export policy decisions.
- `src/lib/lut` and `src/lib/profiles` parse LUTs and resolve profile/catalog
  contracts.
- `packages/luma-color-runtime` is pure TypeScript color math: tone,
  temperature/tint color balance, LUT contracts, transfer/gamut transforms,
  graph resolution, row-band processing, and GLSL helpers.
- `packages/luma-raw-runtime` is the browser RAW metadata/decode/runtime
  boundary, including worker protocol, native artifacts, processed-window facts,
  HDR analysis, fixtures, benchmarks, and native verification.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChrAlpha/LumaForge](https://github.com/ChrAlpha/LumaForge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
