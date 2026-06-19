---
trigger: always_on
description: Portolan is a CLI for publishing and managing **cloud-native geospatial data catalogs**. It orchestrates format conversion (GeoParquet, COG), versioning, and sync to object storage (S3, GCS, Azure)—no running servers, just static files.
---

# Portolan CLI - Development Guide

## What is Portolan?

Portolan is a CLI for publishing and managing **cloud-native geospatial data catalogs**. It orchestrates format conversion (GeoParquet, COG), versioning, and sync to object storage (S3, GCS, Azure)—no running servers, just static files.

**Key concepts:**
- **STAC** (SpatioTemporal Asset Catalog) — The catalog metadata spec
- **GeoParquet** — Cloud-optimized vector data (columnar, spatial indexing)
- **COG** (Cloud-Optimized GeoTIFF) — Cloud-optimized raster data (HTTP range requests)
- **versions.json** — Single source of truth for version history, sync state, and checksums

Portolan doesn't do the heavy lifting—it orchestrates libraries like `geoparquet-io` and `rio-cogeo`.

## Path-scoped rules

Detailed, context-specific guidance lives in `.claude/rules/*.md`. Each file
declares `paths:` globs and Claude Code loads it automatically when you open a
matching file, so this root keeps only always-relevant global facts. Update or
add a rule when a convention changes, the code moves, or a new subsystem needs
its own guidance.

## Terminology (ENFORCED)

**Use STAC terminology exclusively.** Do NOT use "dataset" — it's ambiguous and not part of the STAC spec.

| Term | Meaning | Example |
|------|---------|---------|
| **Catalog** | Root container with metadata | `catalog.json` at repo root |
| **Collection** | Group of related items | `demographics/collection.json` |
| **Item** | Single spatiotemporal entity | `demographics/census-2020/item.json` |
| **Asset** | Actual data file | `demographics/census-2020/data.parquet` |

**Correct:** "Add files to a collection", "Track items", "Push a collection"
**Wrong:** "Add a dataset", "Import datasets", "Dataset management"

## Guiding Principle

AI agents will write most of the code. Human review does not scale to match AI output volume. Therefore: every quality gate must be automated, every convention must be enforceable, and tests must be verified to actually test something.

## Quick Reference

| Resource | Location |
|----------|----------|
| **Roadmap** | [GitHub Issues](https://github.com/portolan-sdi/portolan-cli/issues?q=label%3Aroadmap%3Amvp%2Croadmap%3Anext%2Croadmap%3Afuture) |
| Contributing guide | `docs/contributing.md` |
| Architecture | `pyproject.toml` [tool.importlinter] + [ADR-0025](context/shared/adr/0025-architecture-as-code.md) |
| CI/CD documentation | `context/shared/documentation/ci.md` |
| **Real-world test fixtures** | `context/shared/documentation/test-fixtures.md` |
| ADRs | `context/shared/adr/` |
| Plans & research | `context/shared/` |

**Target Python version:** 3.10+ (matches geoparquet-io dependency)

**CLI entry point:** `portolan` → `portolan_cli:cli` (defined in pyproject.toml)

### ADR Index

| ADR | Decision |
|-----|----------|
| [0001](context/shared/adr/0001-agentic-first-development.md) | Agentic-first: automate all quality gates, TDD mandatory |
| [0002](context/shared/adr/0002-click-for-cli.md) | Click for CLI framework |
| [0003](context/shared/adr/0003-plugin-architecture.md) | Plugin architecture for formats (GeoParquet/COG core, others optional) |
| [0004](context/shared/adr/0004-iceberg-as-plugin.md) | ~~Iceberg as plugin~~ Superseded by ADR-0046 |
| [0005](context/shared/adr/0005-versions-json-source-of-truth.md) | versions.json as single source of truth |
| [0006](context/shared/adr/0006-remote-ownership-model.md) | Portolan owns bucket contents (no external edits) |
| [0007](context/shared/adr/0007-cli-wraps-api.md) | CLI wraps Python API (all logic in library layer) |
| [0008](context/shared/adr/0008-pipx-for-installation.md) | pipx for global installation, uv for development |
| [0009](context/shared/adr/0009-output-dry-run-and-verbose-modes.md) | Dry-run and verbose modes in output functions |
| [0010](context/shared/adr/0010-delegate-conversion-validation.md) | Delegate conversion/validation to upstream libraries |
| [0011](context/shared/adr/0011-mvp-validation-framework.md) | MVP validation framework for format handlers |
| [0012](context/shared/adr/0012-flat-catalog-hierarchy.md) | Flat catalog hierarchy (no nested collections) |
| [0013](context/shared/adr/0013-gitingest-auto-fetch.md) | Auto-fetch dependency docs via gitingest |
| [0014](context/shared/adr/0014-accept-non-cloud-native-formats.md) | Accept non-cloud-native formats with warnings |
| [0015](context/shared/adr/0015-two-tier-versioning-architecture.md) | Two-tier versioning: simple MVP + `[iceberg]` extra for enterprise |
| [0016](context/shared/adr/0016-scan-before-import.md) | Scan-before-import: separate validation from import (like ruff check/fix) |
| [0017](context/shared/adr/0017-mtime-heuristics-change-detection.md) | MTIME + heuristics for change detection (fast gate, O(1) metadata check) |
| [0018](context/shared/adr/0018-metadata-generation-tiers.md) | Metadata generation tiers: auto-extractable → derivable → defaults → human-enrichable |
| [0019](context/shared/adr/0019-cog-optimization-defaults.md) | COG defaults: DEFLATE, predictor=2, 512×512 tiles, nearest resampling |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [portolan-sdi/portolan-cli](https://github.com/portolan-sdi/portolan-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
