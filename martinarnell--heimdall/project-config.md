---
trigger: always_on
description: Compact geocoder built on FSTs. Nominatim-compatible API. Single Rust binary, no runtime dependencies. Currently supports 192 countries. 24 with official government address data (custom importers with CRS conversion where needed), 163 via Photon extracts, 5 via OSM PBF only (Pacific micro-states without Photon coverage). Per-country normalizer configs in `data/normalizers/` (66 files covering all language families) with language-specific phonetic encoding, script-aware normalization (Cyrillic tr
---

# Heimdall — Development Guide

## What this is

Compact geocoder built on FSTs. Nominatim-compatible API. Single Rust binary, no runtime dependencies. Currently supports 192 countries. 24 with official government address data (custom importers with CRS conversion where needed), 163 via Photon extracts, 5 via OSM PBF only (Pacific micro-states without Photon coverage). Per-country normalizer configs in `data/normalizers/` (66 files covering all language families) with language-specific phonetic encoding, script-aware normalization (Cyrillic transliteration, Arabic tashkeel stripping, CJK fullwidth conversion, Hebrew niqqud stripping, Thai tone mark handling, Devanagari matra stripping). Automated rebuild pipeline (`heimdall-build rebuild`) downloads sources, detects changes, and rebuilds affected indices.

## Architecture

```
Manual:   OSM PBF → 3-pass extract → Parquet → enrich (admin hierarchy) → pack (FSTs + record stores)
                                                                         → pack_addr (street-grouped addresses)

Rebuild:  sources.toml → change detect → download → [extract → merge national → merge places_source → merge photon → enrich → pack]
          (per-country, wave-scheduled by RAM budget)
```

Query pipeline: normalizer → FST exact → FST phonetic → Levenshtein → fuzzy layers

Address pipeline: parse query → detect street+number+city → FST range scan → nearest to city coord

## Crate structure

```
crates/
  heimdall-core/       Types, FST index, record store, addr_store, reverse geocoding, NodeCache trait
  heimdall-build/      OSM extraction, enrich, pack, address pack, SSR GML parser, benchmark, rebuild pipeline
  heimdall-normalize/  Per-language text normalization, phonetic encoding, known_variants from TOML
  heimdall-nn/         Neural fuzzy layer stub (not implemented)
  heimdall-api/        Axum HTTP server, multi-country routing, Nominatim-compatible endpoints
  heimdall-compare/    Benchmark framework: 5 query categories, JSONL generation, SQLite results, reports (see crates/heimdall-compare/CLAUDE.md)
```

## Common commands

```bash
# Build everything
cargo build --release -p heimdall-build -p heimdall-api

# Build Sweden index (needs sweden-latest.osm.pbf in data/osm/)
cargo run --release -p heimdall-build -- build \
  --input data/osm/sweden-latest.osm.pbf \
  --output data/index-se

# Rebuild index (skip OSM extraction, reuse existing Parquet)
cargo run --release -p heimdall-build -- build \
  --input data/osm/sweden-latest.osm.pbf \
  --output data/index-se --skip-extract

# Run server (multi-country)
cargo run --release -p heimdall-api -- \
  --index data/index-se \
  --index data/index-no \
  --index data/index-dk \
  --index data/index-fi \
  --index data/index-de \
  --index data/index-gb \
  --index data/index-nz

# ── Benchmark vs Nominatim ─────────────────────────────────────────────

# Generate benchmark queries (5 categories, JSONL with metadata)
cargo run --release -p heimdall-compare -- generate-queries \
  --index data/index-se --index data/index-no \
  --count 10000 --seed 42 --output queries.jsonl

# Run benchmark (1 rps against public Nominatim, resumable)
cargo run --release -p heimdall-compare -- run \
  --queries queries.jsonl --rps 1 --output results.sqlite

# Generate report (console or markdown)
cargo run --release -p heimdall-compare -- report --db results.sqlite
cargo run --release -p heimdall-compare -- report --db results.sqlite --output accuracy.md

# Browse conflicts
cargo run --release -p heimdall-compare -- conflicts --db results.sqlite --min-distance 2000

# Continuous mode (legacy, samples from indices)
cargo run --release -p heimdall-compare -- continuous \
  --index data/index-se --rps 1

# Old benchmark commands (deprecated, use heimdall-compare instead)
cargo run --release -p heimdall-build -- gen-queries --index data/index-se --output queries.txt
cargo run --release -p heimdall-build -- bench --queries queries.txt

# Audit address data in a PBF
cargo run --release -p heimdall-build -- addr-audit --input data/osm/sweden-latest.osm.pbf

# Merge SSR place names into Norway (784K places from Kartverket)
cargo run --release -p heimdall-build -- merge-ssr \
  --index data/index-no \
  --gml data/norway/ssr_extract/Basisdata_0000_Norge_4258_stedsnavn_GML.gml

# Check index stats
cargo run --release -p heimdall-build -- stats --index data/index-se

# Import G-NAF addresses for Australia
cargo run --release -p heimdall-build -- gnaf-import \
  --index data/index-au \
  --gnaf-zip data/downloads/g-naf.zip

# Import NAR addresses for Canada
cargo run --release -p heimdall-build -- nar-import \
  --index data/index-ca \
  --nar-zip data/downloads/NAR_CAN.zip

# Import LINZ addresses for New Zealand
cargo run --release -p heimdall-build -- linz-import \
  --index data/index-nz \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martinarnell/heimdall](https://github.com/martinarnell/heimdall) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
