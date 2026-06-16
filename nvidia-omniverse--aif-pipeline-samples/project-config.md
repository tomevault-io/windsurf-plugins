---
trigger: always_on
description: AIF (AI Factory) digital twin profile for USD assets. Covers AIF-specific metadata, connection points, equipment templates, payload structure, and optimization strategy.
---


# AIF Digital Twin Profile

This profile defines AIF-specific conventions that go beyond SimReady core requirements. It builds on the quality rules in `.cursor/rules/usd-universal.mdc` (which cover core quality and SimReady compliance).

**What's here vs universal rules:**
- Universal rules cover geometry quality, material quality, and SimReady compliance (Z-up, meters, single root, origin, etc.)
- This file covers AIF-only concerns: metadata (`aif:core:*`, `aif:spec:*`), connection points, equipment templates, payload structure, and AIF optimization strategy

## Agent Behavior

### Metadata Requests

When a user asks about AIF metadata or equipment properties:

1. **Determine equipment type** by asking what equipment they are working with:
   - CDU (Coolant Distribution Unit) - 81 properties
   - CRAH (Computer Room Air Handler) - 51 properties
   - UPS (Uninterruptible Power Supply) - 51 properties
   - GB300 Rack - 28 properties (pre-filled with NVIDIA values)
2. **Guide through the workflow:** create template, edit JSON, apply to USD, compose as sublayer, validate.
   - CLI commands are in `.cursor/rules/aif-pipeline-cli.mdc` under Metadata.
3. **After applying metadata,** validate with: `uv run --directory oav validate --rule AIFMetadataChecker <asset>`

### Connection Point Requests

When a user asks about connection points:

1. Confirm optimization is complete first - connection points should be authored against final geometry.
2. Walk through the connection point workflow in the section below.
3. Key naming conventions must match: `<vendor>_<type>_<subtype>_<N>` (e.g., `vertiv_liq_supply_1`).
4. Connection point prims must have `Purpose = guide` and be saved as `<AssetName>_ConnectionPoints.usd`.

### AIF Validation Failures

When `AIFMetadataChecker` or other AIF-specific rules fail:

- **"No properties sublayer found"** - Create and compose a `*_Properties.usda` sublayer using the metadata workflow above.
- **"Missing required attributes"** - Check which `aif:core:*` attributes are missing from the required list below, update the metadata JSON, and re-apply.
- **"Equipment-specific validation failed"** - The `aif:core:assetClass` value does not match the `aif:spec:*` properties present. Ensure the spec properties match the template for the declared asset class.
- **`AIFHierarchyHasRootChecker` failed** - Multiple root prims exist; restructure so there is a single root (excluding `/Render`).
- **`AIFRootIsXformableChecker` failed** - Default prim is not an Xform type; change it to `UsdGeom.Xform`.
- **`AIFAssetAtOriginChecker` failed** - Root prim has a non-identity transform; zero out translation/rotation/scale.

## AIF Hierarchy Structure

A complete AIF asset composes these layers:

```
/<DefaultPrim>              (Xform, equipment root)
├── <Geometry>              (Xform or Mesh prims - CAD-converted geometry)
├── ConnectionPoints/       (Scope - thermal, electrical, airflow interfaces)
│   ├── <vendor>_liq_supply_*     (Plane/Disk, Purpose=guide)
│   ├── <vendor>_liq_return_*     (Plane/Disk, Purpose=guide)
│   ├── <vendor>_electrical_*     (Plane/Disk, Purpose=guide)
│   ├── <vendor>_airvent_intake_* (Plane/Disk, Purpose=guide)
│   └── <vendor>_airvent_outflow_*(Plane/Disk, Purpose=guide)
└── [sublayers]
    ├── <Model>_Properties.usda   (AIF metadata layer)
    └── <Model>_ConnectionPoints.usd (connection point layer)
```

## Metadata Properties

AIF metadata uses two namespaces applied as a separate USDA property layer.

### `aif:core:` — Common Properties (all equipment)

Applied to the equipment root prim. Key properties:

| Property | Type | Description |
|---|---|---|
| `aif:core:manufacturer` | string | Equipment manufacturer name |
| `aif:core:modelNumber` | string | Equipment model number |
| `aif:core:overallGeometryDimensions` | float3 | Overall geometry W x D x H (mm) |
| `aif:core:weight` | float | Weight in kilograms |
| `aif:core:height` / `width` / `depth` | float | Individual dimensions in mm |
| `aif:core:assetClass` | string | Class of AI Factory equipment |
| `aif:core:assetVersion` | string | Design revision of digital twin asset |
| `aif:core:assetCreationDate` | string | ISO 8601 date (YYYY-MM-DD) |
| `aif:core:assetDescription` | string | Human-readable description |
| `aif:core:sceneOptimizerVersion` | string | SO version (tool-managed, excluded from validation) |
| `aif:core:assetValidatorVersion` | string | Validator version (tool-managed, excluded from validation) |

All numeric values use SI units (meters, kilograms, Kelvin, watts) unless noted.

### `aif:spec:` — Equipment-Specific Properties

Vary by equipment type. Create templates with:

```bash
aif-pipeline metadata create --type <type> --output <file>.json
```

| Type | Description | Total Properties |
|------|-------------|-----------------|
| `cdu` | Coolant Distribution Unit | 81 (20 common + 61 specific) |
| `crah` | Computer Room Air Handler | 51 (20 common + 31 specific) |
| `ups` | Uninterruptible Power Supply | 51 (20 common + 31 specific) |
| `gb300_rack` | NVIDIA DGX GB300 Rack | 28 (20 common + 8 specific, pre-filled) |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-Omniverse/aif-pipeline-samples](https://github.com/NVIDIA-Omniverse/aif-pipeline-samples) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
