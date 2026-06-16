---
trigger: always_on
description: How to compose Scene Optimizer preset JSON files. Covers operation catalog, ordering, Python processors, and parameter guidance.
---


# Scene Optimizer Preset Composition

Scene Optimizer presets are JSON arrays of operation objects that define a processing pipeline for USD assets. This guide covers how to compose them.

## Agent Behavior

When a user asks to create or modify a preset:

1. **Start from the canonical template** - read `so/generic/generic_preset.json` as a baseline rather than building from scratch.
2. **Ask what problems they are solving** - map their issues to operations using the catalog below.
3. **Follow operation ordering strictly** - the ordering rules in this file prevent data corruption (e.g., running `deduplicateGeometry` before splitting GeomSubsets produces wrong results silently).
4. **Check vendor presets first** - for vendor equipment, look in `so/vertiv/`, `so/spt/`, `so/trane/` for existing presets before creating a new one.
5. **Prefer external scripts** - when adding `pythonScript` operations, use the library pattern (loading from `so/generic/lib/`) over base64-embedded scripts for maintainability.

When a user asks to fix a specific validation failure with a preset:

1. Look up the fix operation in `.cursor/rules/usd-universal.mdc` (Quick Rule-to-Operation Lookup table).
2. Create a minimal preset with just the needed operations, respecting the ordering rules below.
3. If only one or two operations are needed, a targeted preset is better than running the full generic preset.

## Preset Structure

A preset is a JSON array where each element is an operation:

```json
[
    { "operation": "editStageMetrics", "metersPerUnit": 1.0, "upAxis": 2 },
    { "operation": "meshCleanup", "paths": [], "mergeVertices": true },
    { "operation": "generateNormals", "paths": [], "sharpnessAngle": 60.0 }
]
```

Each operation object must have an `"operation"` key. Additional keys are operation-specific parameters. The full parameter reference is in `so_operations.json` (97KB).

## Operation Catalog

### Stage Operations

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `editStageMetrics` | Set units, up-axis, collapse xforms | `metersPerUnit`, `upAxis` (2=Z), `collapseXforms`, `ignoreKitCameras` |

### Geometry Cleanup

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `meshCleanup` | Fix topology issues | `mergeVertices`, `tolerance`, `removeDegenerateFaces`, `removeDuplicateFaces`, `removeIsolatedVertices`, `makeManifold`, `contractDegenerateEdges`, `mergeBoundaries`, `mergeNeighbors` |
| `generateNormals` | Generate surface normals | `sharpnessAngle` (degrees), `replaceExisting`, `binding` (0=vertex), `weightmode`, `gpuThreshold` |
| `computeExtents` | Compute bounding extents | `paths` |
| `removeSmallGeometry` | Remove tiny geometry | `removeMethod`, `detectionMethod`, `threshold` |
| `manifoldMeshes` | Make meshes watertight | `paths` |

### Geometry Optimization

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `decimateMeshes` | Reduce polygon count | `reductionFactor`, `maxMeanError`, `pinBoundaries`, `allowCutAndGlue`, `cpuVertexCountThreshold`, `gpuVertexCountThreshold`, `guideDecimation` |
| `deduplicateGeometry` | Instance identical meshes | `tolerance`, `duplicateMethod` (2=hierarchy), `fuzzy`, `allowScaling`, `considerDeepTransforms`, `useGpu`, `ignoreAttributes` |
| `merge` | Merge meshes into one | `paths` |
| `remeshMeshes` | Remesh geometry | `paths` |
| `triangulateMeshes` | Convert to triangles | `paths` |
| `subdivideMeshes` | Subdivide surfaces | `paths` |
| `diceMeshes` | Subdivide/dice geometry | `paths` |
| `splitMeshes` | Split meshes by criteria | `paths` |
| `boxClip` | Clip meshes by box | `paths` |

### Hierarchy Operations

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `pruneLeaves` | Remove empty leaf nodes | `pruneMode` (1=empty), `filterInactive` |
| `flattenHierarchy` | Flatten prim hierarchy | `paths` |
| `findFlatHierarchies` | Detect flat hierarchies | `paths` |
| `pivot` | Adjust pivot points | `paths` |

### Material Operations

| Operation | Purpose | Key Parameters |
|-----------|---------|----------------|
| `optimizeMaterials` | Deduplicate/consolidate materials | `optimizeMaterialsMode` (0=consolidate, 2=deduplicate), `materialsPath`, `analysisCheckPrimvars` |
| `optimizePrimvars` | Clean up primvar data | `mode`, `simplify`, `removeIfBound`, `primvars`, `primvarPaths` |

### Analysis Operations (non-destructive)

| Operation | Purpose |
|-----------|---------|
| `findCoincidingMeshes` | Identify overlapping meshes |
| `findHiddenMeshes` | Locate hidden geometry |
| `fitPrimitives` | Fit primitive shapes to meshes |

### Other Operations

| Operation | Purpose |
|-----------|---------|
| `pythonScript` | Run custom Python code |
| `removeAttributes` | Remove prim attributes |
| `generateAtlasUVs` | Generate texture atlas UVs |
| `generateProjectionUVs` | Generate projected UVs |
| `organizePrototypes` | Organize instanced prototypes |
| `optimizeSkelRoots` | Optimize skeleton roots |
| `optimizeTimeSamples` | Reduce time samples |
| `primitivesToMeshes` | Convert primitives to meshes |
| `utilityFunction` | Execute utility functions |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
