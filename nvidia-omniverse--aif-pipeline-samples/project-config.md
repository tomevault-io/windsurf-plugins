---
trigger: always_on
description: USD quality rules for diagnosing and fixing asset issues. Covers core geometry/material quality, SimReady compliance, and the full validation rule catalog.
---


# USD Quality Rules

Three tiers: **Core Quality** (valid, clean USD), **SimReady Compliance** (standardized assets per the SimReady spec), and **AIF-specific** (covered in `.cursor/rules/usd-aif-profile.mdc`).

## Agent Behavior

When a user reports a USD quality issue or a validation rule failure:

1. **Identify the rule name** from their validation output (e.g., `NormalsExistChecker`, `ExtentsChecker`).
2. **Check the Quick Lookup** table below first for the most common failures.
3. **For less common rules,** scan the Symptom-to-Fix tables in the sections that follow.
4. **If multiple fixes are needed,** compose them into a preset respecting the operation ordering in `.cursor/rules/scene-optimizer-presets.mdc`.
5. **Re-validate** after applying fixes to confirm resolution.
6. If a rule is not in these tables, check `.cursor/rules/usd-issues-catalog.mdc` for known patterns.

### Quick Rule-to-Operation Lookup

For the most common validation failures, here are direct fixes:

| Failed Rule | Fix Operation | Key Parameters |
|---|---|---|
| `NormalsExistChecker` / `AIFNormalsValidChecker` | `generateNormals` | `sharpnessAngle: 60.0, replaceExisting: true` |
| `NormalsWindingsChecker` | `generateNormals` | `replaceExisting: true` |
| `ExtentsChecker` / `ZeroExtentChecker` | `computeExtents` | `paths: []` (all prims) |
| `UpAxisZChecker` / `AIFMetersPerUnitChecker` | `editStageMetrics` | `upAxis: 2, metersPerUnit: 1.0` |
| `ValidateTopologyChecker` | `meshCleanup` | `removeDegenerateFaces: true, contractDegenerateEdges: true` |
| `UsdDanglingMaterialBinding` | pythonScript | `validate_fix_material_binding_api.py` |
| `AIFAssetAtOriginChecker` | pythonScript | `transform_stage.py` with identity transform |
| `AIFHierarchyHasRootChecker` | pythonScript | `group.py` to create single root |
| `AIFMetadataChecker` | Not an SO fix | See metadata workflow in `.cursor/rules/usd-aif-profile.mdc` |
| `AIFGeomShallBeMeshChecker` | `primitivesToMeshes` | Convert non-mesh geometry to meshes |

## Symptom → Fix Reference

Use these tables to map a problem to its Scene Optimizer fix. For the full validation rule catalog, see the end of this file.

### Geometry Issues

| Symptom | Detection Rule | Fix (SO Operation) | Parameters |
|---|---|---|---|
| Degenerate/zero-area faces | `ZeroAreaFacesChecker`, `ValidateTopologyChecker` | `meshCleanup` | `removeDegenerateFaces: true, contractDegenerateEdges: true` |
| Non-manifold edges | `NonManifoldChecker` | `meshCleanup` | `makeManifold: true` |
| Duplicate faces | `DuplicateFaceChecker` | `meshCleanup` | `removeDuplicateFaces: true` |
| Isolated/unreferenced vertices | `IsolatedVerticesChecker`, `UnusedMeshTopologyChecker` | `meshCleanup` | `removeIsolatedVertices: true` |
| Colocated vertices | `ColocatedVerticesChecker` | `meshCleanup` | `mergeVertices: true, tolerance: 0.0` |
| Missing normals | `NormalsExistChecker` | `generateNormals` | `sharpnessAngle: 60.0, replaceExisting: true` |
| Invalid normals (NaN/Inf) | `AIFNormalsValidChecker` | `generateNormals` | `replaceExisting: true` |
| Inconsistent winding | `NormalsWindingsChecker`, `WindingsChecker` | `generateNormals` | `replaceExisting: true` |
| Zero/missing extents | `ExtentsChecker`, `ZeroExtentChecker` | `computeExtents` | `paths: []` (all prims) |
| High vertex count | `HighVertexCountChecker` | `decimateMeshes` | `maxMeanError: 0.0001, pinBoundaries: true` |
| Subdivision scheme undefined | `SubdivisionSchemeChecker` | Set explicitly | USD defaults to Catmull-Clark if not set |
| Non-indexed primvars | `IndexedPrimvarChecker` | `optimizePrimvars` | Use indexed format for storage optimization |
| Unused primvars/UVs | `UnusedPrimvarChecker`, `UnusedUVsChecker` | `optimizePrimvars` | `mode: 1, simplify: true` |
| Coinciding/overlapping meshes | `CoincidingGeometryChecker` | pythonScript: `remove_coinciding_meshes.py` | tolerance=0.001 |
| Occluded meshes | `OccludedMeshesChecker` | Review manually or remove | Optional GPU-accelerated analysis |
| Invisible prims | `InvisiblePrimsChecker` | Deactivate instead of hide | Invisible prims still consume resources |
| Sparse meshes needing splitting | `SparseMeshChecker` | `splitMeshes` or `diceMeshes` | Needs dicing, splitting, or clustering |
| Meshes replaceable by primitives | `PrimitiveFitChecker` | `fitPrimitives` | Replace with USD primitive prims |
| Points precision error | `PointsPrecisionErrorChecker` | Fix source data | Points must have precision for <1.0 unit increments |
| Extreme extents (>2^38) | `AlmostExtremeExtentChecker` | Fix transforms/scale | RTX limit is 2^40 |
| Meshes with GeomSubsets blocking dedup | Manual analysis | pythonScript: `split_non_composed_by_geom_subsets.py` | **Must run before dedup** |

### Material Issues

| Symptom | Detection Rule | Fix (SO Operation) | Parameters |
|---|---|---|---|
| Dangling material bindings | `UsdDanglingMaterialBinding` | pythonScript: `validate_fix_material_binding_api.py` | Checks 4 relationship types |
| Missing MaterialBindingAPI | `UsdMaterialBindingApi` | pythonScript: `validate_fix_material_binding_api.py` | Auto-applies API schema |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
