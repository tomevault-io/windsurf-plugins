---
trigger: always_on
description: Known USD issue patterns encountered during CAD-to-USD asset processing. Reference when troubleshooting.
---


# USD Issues Catalog

Known issue patterns encountered during CAD-to-USD asset processing. Each entry is tagged by tier: `[core]` applies to all SimReady assets, `[aif]` is specific to AI Factory digital twin workflows.

This catalog grows over time. When you encounter a new pattern, add it here following the template below.

## Agent Behavior

When troubleshooting an issue:

1. **Check this catalog first** - scan for a matching symptom in the entries below.
2. If it matches, follow the documented Fix and Prevention steps.
3. If the issue is a validation rule failure, cross-reference with `.cursor/rules/usd-universal.mdc` symptom-to-fix tables for the specific rule-to-operation mapping.
4. If the issue is an infrastructure/runtime error (Kit not found, timeout, OOM), see the "Runtime and Infrastructure Errors" section at the bottom of this file.

---

### [core] Dangling material bindings after CAD conversion

**Symptom:** Render errors, materials appear missing, OAV warnings about unresolved material paths.

**Root cause:** CAD converters create material references that can become invalid after hierarchy restructuring or deduplication. Material prims may be removed while bindings on meshes still reference them.

**Detection:** OAV validation, `optimizeMaterials` analysis mode, visual inspection (meshes render with default grey material).

**Fix:** `optimizeMaterials` operation with `optimizeMaterialsMode: 2` (deduplicate), or the `validate_fix_material_binding_api.py` library script to repair the binding API itself.

**Prevention:** Always run `optimizeMaterials` after any hierarchy restructuring. Place material operations after geometry dedup in presets.

---

### [core] Missing normals after CAD conversion

**Symptom:** Faceted rendering, OAV `NormalsExistChecker` failure, visual artifacts on smooth surfaces.

**Root cause:** Many CAD formats (STEP, JT) do not export surface normals. The CAD converter produces meshes without normal attributes.

**Detection:** OAV `NormalsExistChecker`, `AIFNormalsValidChecker`. Visual: surfaces look faceted when they should be smooth.

**Fix:** `generateNormals` operation with `sharpnessAngle: 60.0, replaceExisting: true`. Adjust sharpness angle for sharper (lower value) or smoother (higher value) results.

**Prevention:** Always include `generateNormals` in presets. Place it after `decimateMeshes` since decimation changes geometry.

---

### [core] Duplicate hierarchy branches from CAD assemblies

**Symptom:** Identical subtrees repeated in the prim hierarchy (for example, 48 identical rack units in a server rack). High prim count, slow load times.

**Root cause:** CAD assembly files contain multiple instances of the same component, but the converter expands them into full separate hierarchies rather than using USD instancing.

**Detection:** Visual inspection of hierarchy tree - look for repeated display names. High prim/mesh counts relative to unique geometry.

**Fix:** The `deduplicate_hierarchies_by_display_name.py` library script (from `so/generic/lib/`) identifies and deduplicates matching branches. For mesh-level dedup, use the `deduplicateGeometry` operation with `duplicateMethod: 2, fuzzy: true`.

**Prevention:** Run hierarchy deduplication early in the preset pipeline (before mesh-level operations).

---

### [core] Wrong stage metrics (Y-up or non-meter units)

**Symptom:** Assets appear sideways or at wrong scale. OAV `UpAxisZChecker` or `AIFMetersPerUnitChecker` failure.

**Root cause:** CAD tools commonly use Y-up orientation and millimeter or inch units. The converter may preserve source metrics.

**Detection:** OAV validation. Visual: asset is rotated 90 degrees or appears tiny/enormous.

**Fix:** `editStageMetrics` operation with `upAxis: 2` (Z-up) and `metersPerUnit: 1.0`.

**Prevention:** Always make `editStageMetrics` the first operation in every preset.

---

### [core] Zero or missing extents on boundable prims

**Symptom:** OAV `ExtentsChecker` failure. Bounding box queries return incorrect results, selection and framing in viewport may fail.

**Root cause:** Extents are not automatically recomputed after geometry operations (decimation, cleanup, dedup). Some CAD converters omit them entirely.

**Detection:** OAV `ExtentsChecker`.

**Fix:** `computeExtents` operation with `paths: []` (all prims).

**Prevention:** Always make `computeExtents` the last operation in every preset.

---

### [core] Degenerate geometry surviving conversion

**Symptom:** Visual artifacts, validation warnings about degenerate faces or non-manifold edges. Possible crashes during decimation.

**Root cause:** CAD models often contain construction geometry, zero-area faces, or self-intersecting geometry that persists through conversion.

**Detection:** OAV `ValidateTopologyChecker`. The `meshCleanup` operation reports statistics on what it fixed.

**Fix:** `meshCleanup` with `removeDegenerateFaces: true, contractDegenerateEdges: true, removeIsolatedVertices: true, removeDuplicateFaces: true`. Follow with `removeSmallGeometry` for sub-threshold remnants.

**Prevention:** Always run `meshCleanup` before `decimateMeshes` in presets - decimation on degenerate input can produce worse results.

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
