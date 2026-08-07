---
trigger: always_on
description: PSGC (Philippine Standard Geographic Code) snapshot processor. Converts government Excel/PDF snapshots into RDF linked data (W3C ORG ontology) and matches NAMRIA shapefile boundaries with PSGC codes to produce **hierarchical per-class GeoJSON** — the curated end product — plus enriched and raw stages as intermediate pipeline outputs.
---

# AGENTS.md — Barangay Boundaries Repository

## What This Is

PSGC (Philippine Standard Geographic Code) snapshot processor. Converts government Excel/PDF snapshots into RDF linked data (W3C ORG ontology) and matches NAMRIA shapefile boundaries with PSGC codes to produce **hierarchical per-class GeoJSON** — the curated end product — plus enriched and raw stages as intermediate pipeline outputs.

## Tech Stack

Python 3.13 | Hatchling build | UV package manager | Click CLI | Pydantic v2 | rdflib | geopandas | OpenAI-compatible LLM client

## CLI Entry Point

`brgybnd` → `barangay_boundaries_repository/cli:cli` (10 Click commands)

## Architecture — Two Parallel Pipelines

### Pipeline A: PSGC → RDF

```
PSGC XLSX snapshots (~/commons/barangay/YYYY-MM-DD/)
  → ingest/scanner.py        — classifies files by regex (datafile, changes, press release, prov_summary)
  → ingest/xlsx_parser.py    — parses PSGC sheet (entities) + changes sheets; normalizes ~50 raw change types to 10 canonical
  → ingest/pdf_parser.py     — OCR text from PDF press releases (kreuzberg)
  → [optional] agent/pipeline.py — LLM batch extraction (structured JSON output, batches of 50 entities / 100 changes)
  → rdf/builder.py           — builds RDF graph (ORG ontology: FormalOrganization for Reg/Prov/City/Mun, OrganizationalUnit for Bgy/SubMun)
  → rdf/delta.py             — compute_delta() detects: creation, deletion, code_change, rename, merger, split, name_correction, transfer
  → Output: YYYY-MM-DD/delta.{ttl,nt,jsonld}
```

### Pipeline B: NAMRIA Shapefiles → Hierarchical Per-Class GeoJSON

```
namria/phl_admbnda_adm{0-4}_*.shp (version 2023-11-06)
  → namria_converter.py         — shapefile → GeoJSON (Douglas-Peucker simplify, default 0.005°)   [stage 1: raw_t0p005/]
  → coverage.py                  — PSGC pcode vs GeoJSON pcode comparison per admin level
  → generate_huc_mapping.py     — builds huc_adm2_mapping.json for HUCs, MM districts, Special Geographic Areas
  → enrich.py                    — adds psgc_id, psgc_code, psgc_name, psgc_status, match_confidence   [stage 2: enriched_t0p005/]
  → classifier.py                — assigns PSGC type/class (barangay, HUC, ICC, component city, …) per feature
  → splitter.py                  — splits enriched features into per-class files
  → hierarchical.py              — orchestrates phases 3–5 (classify → split → report)               [curated: hierarchical_t0p005/]
  → Output: YYYY-MM-DD/hierarchical_t0p005/<class>.geojson + classification_report.json + summary.md
```

### Data tiers

- **Hierarchical** = curated / recommended end product. Per-class GeoJSON files (`barangays`, `provinces`, `municipalities`, `highly_urbanized_cities`, `independent_component_cities`, `component_cities`, `submunicipalities`, `special_geographic_areas`, …) plus `classification_report.json` and `summary.md`. This is what GitHub Releases publish.
- **Enriched** = intermediate enrichment stage. `adm0`–`adm4.geojson` annotated with PSGC `psgc_code`/`psgc_name`/`psgc_status`/`match_confidence`. Output of `enrich.py`; consumed by the classifier.
- **Raw** = intermediate source stage. NAMRIA-converted GeoJSON, pre-enrichment. Output of `namria_converter.py`.

## CLI Commands

| Command | Source | Purpose |
|---------|--------|---------|
| `brgybnd list` | cli.py:34 | List PSGC snapshots |
| `brgybnd ingest --date YYYY-MM-DD` | cli.py:46 | Parse datafile + changes + press release |
| `brgybnd process --date YYYY-MM-DD [--full] [--format turtle\|json-ld\|nt]` | cli.py:83 | Build full RDF or delta |
| `brgybnd process-all` | cli.py:158 | Generate all deltas |
| `brgybnd delta --from DATE --to DATE` | cli.py:234 | Delta between two dates |
| `brgybnd convert-geo [--levels] [--tolerance]` | cli.py:262 | Shapefile → GeoJSON |
| `brgybnd validate --input FILE` | cli.py:323 | Validate RDF + ORG conformance |
| `brgybnd coverage --date DATE` | cli.py:353 | PSGC vs GeoJSON coverage |
| `brgybnd enrich --date DATE` | cli.py:515 | Enrich GeoJSON with PSGC IDs |
| `brgybnd build-hierarchical --date YYYY-MM-DD [--skip-convert] [--skip-enrich]` | cli.py:566 | Run convert → enrich → classify → split, writes `hierarchical_t0p005/` (curated per-class output) |

## Key Data Model (models/schemas.py)

- **GeographicEntity**: 10-digit PSGC code (pattern `^\d{10}$`), name, level (Reg/Prov/City/Mun/Bgy/SubMun), correspondence_code, optional city_class, income_class, population, is_capital, is_poblacion
- **ChangeEvent**: event_type (creation/deletion/transfer/renaming/merger/split/reclassification/code_change/reenlistment/unknown), entity_name, new_code, old_code, legal_basis, effective_date, plebiscite_date, mother_unit
- **RdfTriple**: subject_uri, predicate_uri, object_value, datatype, lang
- **BatchExtractionResult**: entities + change_events lists

## PSGC Code Structure

10-digit hierarchical: `PPCCMMLLLL0`
- `PP00000000` — Region
- `PPCCC00000` — Province
- `PPCCCMMLL0` — Municipality/City
- `PPCCCMMLLL` — Barangay (full 10 digits)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bendlikeabamboo/barangay-boundaries-repository](https://github.com/bendlikeabamboo/barangay-boundaries-repository) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
