---
trigger: always_on
description: Generates Unity assets and reports Unity-specific requirements.
---

# AGENTS.md — Miku Engineering Constitution

## 1. Project Identity

Miku is a production-quality, public open-source project.

Its primary purpose is:

Blender 5.2 Shader Nodes
→ target-neutral Miku semantic IR
→ Unity 6 URP Shader Graph
→ editable `.shadergraph` and `.shadersubgraph` assets

Miku is not a disposable prototype, code-generation experiment, or demo.

Every contribution must prioritize:

1. Correctness
2. Deterministic behavior
3. Data integrity
4. Maintainability
5. Compatibility
6. Clear diagnostics
7. Performance
8. Feature coverage

Do not trade correctness or asset safety for faster implementation.

The current primary target is:

- Blender 5.2
- Unity 6
- URP
- Shader Graph
- Editor-time conversion
- Editable Shader Graph assets

Do not generate ShaderLab unless a future approved architecture decision explicitly reintroduces that backend.

---

## 1.1 Canonical Miku 1.x Source Boundary

Miku 1.x feature work must use only these canonical source roots:

- `miku/`
- `miku_blender/`
- `extensions/miku_shader_converter/`
- `unity/Packages/com.miku.shaderconverter/`

The active Blender extension IDs are `miku_semantic_exporter` and
`miku_gpl_bake_worker`. The active Unity package ID is
`com.miku.shaderconverter`.

Before changing Miku 1.x source, preflight the repository root and stop if any
of these markers is missing:

- `miku/`
- `extensions/miku_shader_converter/`
- `unity/Packages/com.miku.shaderconverter/package.json` whose `name` is
  `com.miku.shaderconverter`

Passing this marker check establishes the canonical repository boundary. A
retired B2U checkout, a Unity validation project's embedded
`Packages/com.miku.shaderconverter/`, an installed Blender extension, or a
`dist/` archive must never be selected as the implementation root.

The following paths and identities belong to the retired B2U architecture and
must not receive new Miku features unless a task explicitly authorizes legacy
migration or removal work:

- `b2u_mvp/`
- `b2u_mvp_blender/`
- `addons/b2u_mvp_blender/`
- `unity/Packages/com.b2u.shaderconverter/`

Installed Blender extensions and `dist/` archives are build outputs, not source
of truth. Never patch an installed copy as the implementation. Change the
canonical Miku source, build deterministic packages, install those packages,
and verify the installed module paths and hashes. Unity validation projects and
installed Blender extension copies may only be populated from deterministic
canonical-source builds. Compare their file manifests and SHA-256 hashes with
the canonical build before treating validation results as evidence.

For this repository's Windows validation environment, the Blender installation
root is fixed to:

`C:\SteamLibrary\steamapps\common\Blender`

All Blender launches, headless tests, extension installation, and export
validation must call this executable directly:

`C:\SteamLibrary\steamapps\common\Blender\blender.exe`

Do not use `PATH`, `blender-launcher.exe`, `.tools`, Program Files, or another
Blender installation as a fallback. Before validation, assert
`bpy.app.version == (5, 2, 0)` and fail clearly on any mismatch. Do not overwrite
installed extensions while a Blender GUI process is running or contains
unsaved work; save and close Blender first.

---

## 2. Required Working Process

Before modifying code:

1. Read this file completely.
2. Read the nearest nested `AGENTS.md`, if one exists.
3. Inspect the repository structure.
4. Read the relevant architecture and compatibility documents.
5. Search for existing implementations before creating new abstractions.
6. Inspect existing tests and fixtures.
7. Check the current Git diff.
8. Identify the exact supported Blender, Unity, URP, and Shader Graph versions.

For a complex feature, cross-module refactor, schema change, compatibility change, or task expected to affect more than one subsystem:

- Create or update an ExecPlan under `docs/plans/`.
- Follow `PLANS.md`.
- Keep the plan updated while implementing.
- Record discoveries, decisions, rejected alternatives, tests, and remaining work.
- Do not let implementation silently diverge from the plan.

Do not stop after producing a plan unless the task explicitly requests planning only. Continue through implementation and validation.

---

## 3. Repository Scope and Ownership

Respect existing repository structure.

Do not create parallel implementations of functionality that already exists.

The intended architectural boundaries are:

- Blender integration:
  Reads Blender data through supported Blender APIs.
  Converts Blender-specific nodes into target-neutral Miku data.

- Core:
  Owns semantic IR, type checking, graph validation, normalization,
  effect recognition, diagnostics, deterministic IDs, and schema handling.
  Core must not depend on UnityEditor or bpy.

- Unity integration:
  Reads target-neutral Miku IR.
  Selects a version-specific Shader Graph backend.
  Generates Unity assets and reports Unity-specific requirements.

- Schemas:
  Owns versioned interchange formats.
  Schema changes require compatibility documentation and tests.

- Tests:
  Owns unit, integration, snapshot, fixture, and compatibility tests.

- Documentation:
  Owns architecture, contributor, compatibility, schema, and user documentation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GenshinmasterJinHang/Miku-Material-Converter-Blender-to-Unity-](https://github.com/GenshinmasterJinHang/Miku-Material-Converter-Blender-to-Unity-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-05 -->
