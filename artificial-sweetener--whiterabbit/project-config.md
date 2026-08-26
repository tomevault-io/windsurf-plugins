---
trigger: always_on
description: WhiteRabbit is a high-quality, strongly typed ComfyUI extension for video frame
---

# AGENTS.md

## Mission

WhiteRabbit is a high-quality, strongly typed ComfyUI extension for video frame
manipulation, interpolation, scaling, stabilization, looping, and post-processing.
Engineering priorities are behavior safety, strict separation of concerns, complete
feature integrations, deterministic execution, explicit validation, ComfyUI
compatibility, runtime safety, and long-term maintainability.

## Public behavior boundary

- Preserve serialized workflow compatibility unless the maintainer explicitly
  approves a breaking change.
- Treat node identifiers, display names, categories, input keys and order, defaults,
  output types and order, execution return shapes, and user-visible behavior as the
  public contract.
- Change internals freely and completely within that boundary.
- Add characterization tests before structurally changing behavior-critical code.

## Localization policy

- Route every WhiteRabbit-owned visible node label, description, tooltip, output
  label, category, and eligible option label through the canonical v3 schema and
  its locale catalog. Do not create a second English catalog.
- Treat `locales/languages.json` as the sole supported-locale registry. Do not
  duplicate locale inventories in source, tools, tests, or documentation.
- English v3 schemas are the canonical source and fallback language. Every
  release-enabled non-English locale must provide complete direct translations
  for every owned visible schema field in the same change.
- Add or change visible English text and all release-enabled translations
  atomically. Adding a locale requires complete catalog and README coverage.
- Preserve serialized workflow compatibility: node IDs, categories, input IDs,
  option values, output positions, and execution behavior are never translated.
  Locale catalogs translate presentation only.
- Preserve ComfyUI-owned and third-party text as supplied by ComfyUI; do not
  maintain a parallel corpus for text WhiteRabbit does not own.
- Keep each release-enabled localized README structurally and factually aligned
  with `readme.md`, while writing natural, audience-appropriate prose rather
  than mechanical translations.
- Write translations directly. Maintain the locale terminology guide and do not
  weaken locale-registry, coverage, stale-key, or integration validation to
  accept incomplete work.

## Authoritative environment

- Use Windows PowerShell syntax.
- Run all verification from the repository root.
- Use `E:\ComfyUI\.venv`; do not create a repository-local virtual environment.
- Tests: `E:\ComfyUI\.venv\Scripts\python.exe -m pytest -n auto -q`
- Lint: `E:\ComfyUI\.venv\Scripts\ruff.exe check .`
- Format: `E:\ComfyUI\.venv\Scripts\ruff.exe format .`
- Types: `E:\ComfyUI\.venv\Scripts\mypy.exe --strict __init__.py whiterabbit tests`

## Architecture

- `whiterabbit/nodes_v3`: thin Comfy v3 schemas and execution entry points.
- `whiterabbit/services`: application use cases and orchestration.
- `whiterabbit/domain`: stable value objects, plans, policies, and pure behavior.
- `whiterabbit/runtime`: ComfyUI, tensor, model, device, filesystem, image, and
  network adapters.
- `whiterabbit/shared`: small lower-level validation and logging primitives.
- Dependencies flow from nodes to services and from services to domain/runtime.
- Domain modules never import ComfyUI.
- Nodes never own model loading, downloads, filesystem policy, or non-trivial
  algorithms.
- Every concern has one authoritative owner. Do not duplicate policy across layers.
- Complete refactors fully: update all callsites, remove obsolete internal paths,
  and do not leave internal compatibility shims or transitional adapters.
- Preserve compatibility only at ComfyUI-facing and persisted-workflow boundaries.

## ComfyUI nodes

- Comfy v3 is the sole export path.
- The root `comfy_entrypoint()` and `whiterabbit.nodes_v3.get_nodes()` are the
  authoritative registry.
- Keep schemas deterministic and free of expensive IO, network access, or model
  loading.
- Give every node a concise description and every visible input/output an accurate
  tooltip.
- Validate inputs before side effects and raise actionable errors.
- Return exactly the declared output shape.

## RIFE and model behavior

- Resolve model locations through ComfyUI's `frame_interpolation` folder registry.
- WhiteRabbit owns its enhanced RIFE execution behavior, including scale control,
  internal bidirectional ensemble, arbitrary timing, caching, FPS resampling, seam
  analysis, and stabilization.
- Integrate ComfyUI memory management without reducing WhiteRabbit's feature set.
- Automatic downloads are limited to trusted catalog artifacts, use timeouts,
  bounded destinations, temporary files, atomic publication, checksum validation,
  progress reporting, and cleanup on failure.
- Model format detection must fail with an actionable error when unsupported.
- Device, dtype, VRAM, loading, offloading, and cache lifetime are correctness
  concerns and belong to runtime owners.

## Typing

- `mypy --strict` must pass for source and tests.
- Type every function signature and important internal state.
- Prefer dataclasses, enums, `Literal`, `TypedDict`, and `Protocol` over loose
  dictionaries and `Any`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Artificial-Sweetener/WhiteRabbit](https://github.com/Artificial-Sweetener/WhiteRabbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
