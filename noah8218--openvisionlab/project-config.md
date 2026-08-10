---
trigger: always_on
description: This file defines the working agreement for Codex in this repository.
---

# AGENTS.md

This file defines the working agreement for Codex in this repository.

## Work Location

- Primary implementation and verification work starts in `C:\Git\OpenVisionLab_Dev`.
- `C:\Git\OpenVisionLab` is the original OpenVisionLab repository that receives reviewed, stabilized changes from Dev.
- Do not bulk-copy Dev over the original repository. Move changes by reviewed patch, cherry-pick, or import.
- Do not run `git push` unless the user explicitly requests `PUSH`.

## Product Identity

- OpenVisionLab is an OpenCvSharp4-based rule-based vision recipe workbench. Rule-based teaching, execution, drawings, and repeatable validation are the product core.
- LLM support is an optional, maintenance-mode XML authoring aid. It is not the primary product direction and must not be a prerequisite for operating the workbench.
- It is for learning, verifying, and composing image-processing inspection recipes with tools such as Threshold, Blob, Contour, Line/Length, Matching, EdgeBasedMatching, and Feature/Shape-style workflows.
- The main workflow is sample image -> direct PropertyGrid teaching and Pipeline composition -> explicit Preview/Run -> layer/result comparison -> N-sample validation -> saved recipe. Existing LLM XML draft/validation/import may optionally accelerate the composition step.
- It is not a camera, lighting, PLC, or I/O integration platform.

## Localized User Manual Contract

- The application `Guide` command must select the offline manual from
  `OpenVisionLanguageService.CurrentLanguage`: Korean UI opens only the Korean
  manual and English UI opens only the English manual.
- Do not silently fall back to a manual in a different language. A missing,
  damaged, duplicated, or hash-mismatched selected-language entry must fail
  closed with the localized Guide error.
- Package all supported manual languages beside the EXE under `Guide`, and
  validate each exact language/file/SHA-256 mapping through
  `Guide\guide-manifest.json` before opening it.
- Keep Korean and English manuals structurally equivalent: the same supported
  workflow chapters, Tool coverage, reference scope, numbered callouts, and
  explicit Preview/Run contracts. A language-specific manual must use current
  UI captures from that same application language wherever UI text is visible.
- A language change or Guide open must not run Preview/Run, create/delete/select
  layers, change the active layer, or mutate Pipeline routing.
- Verify both language selections from the current build and a copied clean
  runtime. Check exact HTML language markers, manifest hashes, missing-language
  failure, and that switching Korean -> English -> Korean changes only the
  selected manual path and localized presentation state.

## User-Centered Workflow And Persisted Setup

- Design each feature from the operator's goal and the shortest safe normal
  workflow, not from internal screen, component, class, or storage boundaries.
- When one task requires related settings across several views, dialogs, or
  buttons, treat that friction as a design problem. Consolidate settings that
  belong to one durable workflow into one coherent first-use option or setup
  surface.
- Persist reusable setup only after explicit operator confirmation and at the
  narrowest correct Tool, Recipe, project, workspace, or user scope. Restore
  it on the next equivalent use so the operator does not repeat the same setup.
- Keep restored values visible and editable, provide an explicit reset/default
  path, validate them before reuse, and explain stale or incompatible state.
- Do not silently share task-specific settings across unrelated Recipes,
  projects, workspaces, or users. Do not remember destructive,
  security-sensitive, or safety-critical choices implicitly.
- Restoring configuration must not execute Preview/Run, create/delete/select
  layers, change the active layer, or mutate Pipeline routing.
- Verify reusable setup through save, close/reload/reopen, exact restoration,
  visible reset, stale-state rejection where applicable, and zero unintended
  execution/layer/routing side effects.
- Use
  `docs/reports/OPENVISIONLAB_USER_CENTERED_WORKFLOW_DIRECTION_20260729.md`
  as the reusable admission template and future development direction.

## LLM Maintenance Mode And Preserved Evidence

- OpenVisionLab is a deterministic rule-based vision recipe workbench. LLM support is an optional maintenance-mode XML authoring aid and must not be required for normal operation.
- Do not start provider integrations, consumer-web automation, new prompt families, inspection-intent skills, transcript campaigns, or repeated dataset tuning unless the user explicitly reopens a named task.
- Preserve existing LLM Assistant, Guided Setup, XML guide/catalog, validation/import gates, and recorded correction evidence. Fix only a reproduced regression, data-loss risk, invalid XML acceptance, or compatibility break inside that frozen scope.
- Provider transport remains optional. Manual prompt copy/XML paste may remain available; API credentials and consumer-web automation must not become product dependencies.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noah8218/OpenVisionLab](https://github.com/Noah8218/OpenVisionLab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
