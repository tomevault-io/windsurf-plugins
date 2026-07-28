---
trigger: always_on
description: Validates all JSON examples against their corresponding schemas.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repository defines file format specifications for ABAP repository objects. Each ABAP object type (e.g., CLAS, INTF, DOMA) has a corresponding folder in `file-formats/` containing:
- JSON schema definitions (e.g., `clas-v1.json`)
- ABAP interface types in `type/` subfolder (e.g., `zif_aff_clas_v1.intf.abap`)
- Example files in `examples/` subfolder

JSON schemas are **automatically generated** from ABAP interface definitions, not manually written.

## Common Commands

### Linting
```bash
npm run lint
```
Runs abaplint with auto-fix on ABAP interface files in `file-formats/`.

### JSON Schema Generation
```bash
cd generate
npm ci
npm run compile   # Transpiles ABAP to JavaScript
npm run generate  # Generates JSON schemas from ABAP types
npm test          # Runs both compile and generate
```

The generation process:
1. Downloads ABAP generator classes from [SAP/abap-file-formats-tools](https://github.com/SAP/abap-file-formats-tools) via `oras-pull` (OCI artifact from `ghcr.io/sap/abap-file-formats-tools`)
2. Copies ABAP interface files from `file-formats/` to `downport/`
3. Transpiles ABAP to JavaScript in `output/` using `@abaplint/transpiler-cli`
4. Runs `aff.mjs` to generate JSON schemas in `generated/`

The generation logic itself (classes like `zcl_aff_generator`, `zcl_aff_writer_json_schema`, `zcl_aff_abap_doc_reader`) lives in the [abap-file-formats-tools](https://github.com/SAP/abap-file-formats-tools) repository, not in this repo. These classes introspect ABAP type definitions and ABAP Doc annotations to produce JSON Schema output. Alternatively, individuals can clone the tools repo into an ABAP system and run the report `z_generate_json_schema` directly there.

### JSON Validation
```bash
python json-validator.py
```
Validates all JSON examples against their corresponding schemas.

### Compatibility Check
```bash
cd compatibility-check
npm ci
npm run check
```
Checks if JSON schema changes are compatible with previous versions using `json-schema-diff`.

## Architecture

See the full specs in `docs/`:
- `docs/specification.md` — file naming conventions, file extensions, formatting standards
- `docs/json.md` — format versions, compatibility rules, ABAP type interface pattern, ABAP Doc annotations, naming conventions

## Adding a New Object Type

1. Create `file-formats/<objtype>/` directory structure
2. Write ABAP interface `type/zif_aff_<objtype>_v1.intf.abap` with `ty_main` structure
3. Add ABAP Doc comments and annotations
4. Run `cd generate && npm test` to generate JSON schema
5. Copy generated schema from `generate/generated/` to `file-formats/<objtype>/<objtype>-v1.json`
6. Create example files in `examples/` subfolder
7. Run `python json-validator.py` to validate examples
8. Run `npm run lint` to check ABAP code style
9. Create README.md documenting file structure (see `file-formats/clas/README.md` as template)

## abaplint Configuration

The `abaplint.jsonc` file enforces strict ABAP coding standards including:
- Object naming patterns (INTF must match `^ZIF_AFF_[A-Z0-9]{4}_V[0-9]+$`)
- Type naming (`ty_*` prefix required)
- Constant naming (`co_*` prefix required)
- Syntax checks for v702 ABAP version
- Various code quality rules

## GitHub Workflows

CI checks run on pull requests:
- **generate.yml**: Verifies generated schemas match committed versions
- **json-compatibility.yml**: Checks schema changes are compatible
- **editorconfig.yml**: Validates file formatting
- **py-validation.yml**: Validates JSON examples
- **markdown-link-check.yml**: Checks for broken links

## Patterns from Recent PRs

Based on analysis of recent merged PRs, the AFF project follows these patterns:

### Making Incompatible Changes to v1

**When it's acceptable**: If an object type is not yet widely adopted (e.g., not used in abapGit or other major tools), incompatible changes can be made to v1 instead of creating v2. This requires:
1. Explicit confirmation from maintainers that the format is not in use
2. Justification in the PR body (e.g., PR #728: "the SRVD implementation is not used in ABAPGit yet")
3. Acceptance that compatibility checks will fail and need to be overridden

**Pattern observed in PR #726 (DOMA)**: The team discussed whether to create v2 or modify v1. After confirming no known usages, they chose to modify v1 incompatibly, ignoring the compatibility check.

### Adding Default Values to Enums

**When adding defaults** (PR #719 - DDLS):
- Add `$default` annotation to the enum field in the ABAP interface
- The default allows future enum values to be added compatibly
- Systems that don't support new enum values will fall back to the default
- Format version doesn't need to increment

**Example pattern**:
```abap
"! $values {@link zif_aff_ddls_v1.data:co_source_origin}
"! $default {@link zif_aff_ddls_v1.data:co_source_origin.abap_development_tools}
TYPES ty_source_origin TYPE c LENGTH 1.
```

### Making Fields Required

**When to make fields required** (PR #728 - SRVD):
- If the field already has a `$default` annotation, adding `$required` is a compatible change
- The field will now always be serialized (even when it has the default value)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SAP/abap-file-formats](https://github.com/SAP/abap-file-formats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
