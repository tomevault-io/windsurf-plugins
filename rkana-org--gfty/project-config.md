---
trigger: always_on
description: `gfty` is a Rust CLI/package, Nix module, browser designer, and Onshape FeatureScripts for reproducibly turning
---

# gfty development guide

## Scope and purpose

`gfty` is a Rust CLI/package, Nix module, browser designer, and Onshape FeatureScripts for reproducibly turning
SVG label artwork and typed bin TOML into colored Onshape geometry. It supports
standalone bins, individual labels, row-major multi-label plates, and the
Gridfinity Ultimate web workflow.

The current end-to-end model is:

1. Typed bin TOML -> canonical Gridfinity Ultimate configuration JSON.
2. Label TOML + SVG template/icons -> composed SVG.
3. Composed SVG -> compact schema-version-2 geometry JSON grouped by filament.
4. A referenced bin configuration produces the blank prototype in Onshape.
5. `featurescripts/labels/gfty_label_instances.fs` copies that prototype, creates the
   artwork, and joins multi-label output with sacrificial connector plates.
6. Nix builders expose reproducible bin/label/plate bundles and flake-parts
   outputs.

Read the repository `README.md` before changing user-facing behavior. The parent
repository also has `/home/malte/onshape/AGENTS.md`; in particular, local
FeatureScript docs are in `../featurescript-docs/` and standard-library source
is in `../std-library/`.

## Repository map

- `designer/`: Gridfinity Ultimate browser application and Nix site package.
- `featurescripts/`: every maintained FeatureScript, organized by labels,
  configuration, and divider generation.
- `src/main.rs`: command dispatch, output behavior, and inspection.
- `src/cli.rs`: Clap interface. Keep stdout clean for data-producing commands.
- `src/config.rs`: label TOML schema, path resolution, discovery, and validation.
- `src/bin_config.rs`: current bin TOML, divider normalization, canonical model
  JSON conversion, and generated-part discovery.
- `src/component_config.rs`: independent base/rim/swappable-label/set schemas,
  compatibility checks, carrier configurations, and semantic request keys.
- `src/artifact_cache.rs`: verified runtime STEP/PNG cache outside the Nix store.
- `src/create.rs`: unsaved label creation and reusable TOML saving.
- `src/credentials.rs`: protected Onshape credential-file discovery.
- `src/onshape.rs`: signed encode/translate/poll/download API operations.
- `src/step.rs`: expected filament manifest validation and atomic downloads.
- `docs/ARCHITECTURE.md`: current configuration, geometry, Onshape transport,
  cache, Nix-boundary, FeatureScript, and designer contracts.
- `src/template.rs`: SVG template contract and icon-box metadata.
- `src/compose.rs`: shared label composition pipeline.
- `src/color.rs`: SVG fill/stroke discovery, sidecars, overrides, preview colors.
- `src/layout.rs`: icon flow/alignment.
- `src/svg.rs`: reusable `usvg` parser and text outlining.
- `src/export.rs`: schema-version-2 geometry export.
- `src/plate.rs`: dimension-constrained row-major plate layout.
- `src/terminal_preview.rs`: native rasteroid previews.
- `src/watch.rs`: dependency watching and rebuild presentation.
- `featurescripts/labels/gfty_label_instances.fs`: version-2 Onshape importer.
- `featurescripts/configuration/variable_configured_derived.fs`: wrapper around
  native Derived which forwards current variables into source configuration IDs.
- `featurescripts/configuration/extract_json_config.fs`: Gridfinity JSON-to-variable
  bridge.
- `featurescripts/dividers/walls_grid.fs`: divider and easy-grab wall generator.
- `nix/mk-label.nix`, `nix/mk-plate.nix`: passthru builders.
- `flake-module.nix`: typed flake-parts label/plate definitions and nested output
  packages.
- `examples/`: standalone flake-parts integration test and documentation.
- `tests/bin-designer-conformance.js`: verifies the browser default
  configuration against a frozen JSON fixture.
- `.github/workflows/designer-pages.yml`: deploys `packages.designer` from
  `designer-v*` tags.

## Core behavior and invariants

### Configuration schemas and products

Only current, explicitly typed schemas are supported:

| kind | version | generated product |
| --- | ---: | --- |
| `bin` | 2 | exact `Bin` body |
| `base` | 1 | exact `Base` body |
| `rim` | 1 | exact `SwappableRim` body |
| `swappable-label` | 1 | exact `SwappableLabel` body |
| `bin-set` | 1 | declared compatible Gridfinity bodies |
| `label` | 1 | overlapping `part-<filament>` artwork-label bodies |

The standard `ConnectorPin` has no TOML schema. It is exported independently or
included in a bin set. Labels require a bin reference. Do not add compatibility
parsing for retired schemas or implicit kinds.

### Paths and discovery

- There is no required project marker or directory layout.
- Absolute paths work everywhere.
- Paths in saved label TOML resolve relative to that TOML.
- `gfty label create` paths resolve relative to the current working directory.
- Pathless `validate` recursively scans `ROOT/labels`; `--root` overrides ROOT.
- `.svg` icon values are paths. Other icon values are aliases in `[icon.NAME]`.
- Icon color overrides may be declared on `[icon.NAME.colors]` aliases or
  inline on individual `[[icons.BOX]]` placements.

### Templates and composition

- Templates require physical `width`/`height`, a `viewBox`, unique `text-*`
  elements, and unique `icons-*` rectangles.
- `data-gfty-direction` is `horizontal` or `vertical`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rkana-org/gfty](https://github.com/rkana-org/gfty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
