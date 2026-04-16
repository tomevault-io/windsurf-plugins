---
trigger: always_on
description: **Name**: Darwin Operator Symmetry Atlas (DOSA)
---

# CLAUDE.md — Darwin Operator Symmetry Atlas

## Project Identity

**Name**: Darwin Operator Symmetry Atlas (DOSA)
**Version**: 2.0.0-alpha
**Principal Investigator**: Demetrios Chiuratto Agourakis
**Target Publication**: Scientific Data (Nature Portfolio) — Data Descriptor

---

## Mission Statement

Build a reproducible, DOI-versioned database of operator-defined symmetries in complete bacterial replicons, implementing a hybrid Demetrios + Julia architecture with cross-validation between implementations.

**NO PYTHON**. This project uses exclusively Julia (Layers 0-1) and Demetrios (Layer 2).

---

## Architecture Overview

```
Layer 3: Artifacts     → CSV/JSONL/Parquet (Zenodo DOI)
Layer 2: Demetrios     → High-performance kernels with epistemic computing
Layer 1: Julia         → Orchestration, NCBI fetch, validation
Layer 0: Julia Pure    → Reference implementation (fallback, cross-validation)
```

### Why This Architecture?

1. **Demetrios (Layer 2)**: Showcases the language's units of measure, refinement types, and epistemic computing for scientific applications
2. **Julia (Layers 0-1)**: Provides reproducibility guarantee for Scientific Data reviewers who may not have Demetrios installed
3. **Cross-validation**: Ensures both implementations produce **identical** results, catching bugs in either

---

## Directory Structure (Canonical)

```
darwin-atlas/
├── CLAUDE.md                     # THIS FILE - READ FIRST
├── README.md                     # Project documentation
├── Makefile                      # Build orchestration
├── .zenodo.json                  # DOI metadata
│
├── demetrios/                    # Layer 2: Demetrios Kernels
│   ├── demetrios.toml            # Project config
│   ├── src/
│   │   ├── lib.d                 # Library root, exports
│   │   ├── operators.d           # S/R/K/RC definitions with units
│   │   ├── exact_symmetry.d      # Fixed points, orbit ratio
│   │   ├── approx_metric.d       # d_min/L with refinement types
│   │   ├── quaternion.d          # Dic_n lift verification
│   │   └── ffi.d                 # C ABI exports for Julia
│   └── tests/
│
├── julia/                        # Layers 0 + 1
│   ├── Project.toml
│   ├── Manifest.toml             # MUST BE COMMITTED (reproducibility)
│   ├── src/
│   │   ├── DarwinAtlas.jl        # Module root
│   │   ├── Types.jl              # Shared type definitions
│   │   ├── Operators.jl          # Pure Julia operators (Layer 0)
│   │   ├── ExactSymmetry.jl      # Pure Julia exact symmetry
│   │   ├── ApproxMetric.jl       # Pure Julia approx metric
│   │   ├── QuaternionLift.jl     # Pure Julia quaternion
│   │   ├── NCBIFetch.jl          # NCBI download + manifest
│   │   ├── Validation.jl         # Technical validation suite
│   │   ├── DemetriosFFI.jl       # ccall wrappers (Layer 1→2)
│   │   └── CrossValidation.jl    # Demetrios vs Julia comparison
│   ├── test/
│   │   └── runtests.jl
│   └── scripts/
│       ├── run_pipeline.jl
│       └── cross_validation.jl
│
├── data/                         # Layer 3: Outputs
│   ├── raw/                      # Downloaded sequences (gitignored)
│   ├── manifest/
│   │   ├── manifest.jsonl
│   │   └── checksums.sha256
│   └── tables/
│       ├── atlas_replicons.csv
│       ├── atlas_windows_exact.csv
│       ├── approx_symmetry_stats.csv
│       ├── dicyclic_lifts.csv
│       └── quaternion_results.csv
│
├── paper/                        # Scientific Data manuscript
│   ├── main.tex
│   └── figures/
│
└── .github/workflows/ci.yml      # Automated testing
```

---

## Technical Specifications

### Operator Definitions (Mathematical Foundation)

| Symbol | Name | Definition | Group |
|--------|------|------------|-------|
| I | Identity | σ(i) = s_i | D_4 |
| R | Reverse | σ(i) = s_{n-1-i} | D_4 |
| K | Complement | σ(i) = complement(s_i) | D_4 |
| RC | Rev-Comp | σ(i) = complement(s_{n-1-i}) | D_4 |

### Data Schema (Canonical)

#### atlas_replicons.csv
| Field | Type | Constraint |
|-------|------|------------|
| assembly_accession | String | GCF_... format |
| replicon_id | String | Internal stable ID |
| replicon_type | Enum | {chromosome, plasmid, other} |
| length_bp | Int64 | > 0 |
| gc_fraction | Float64 | 0.0 ≤ x ≤ 1.0 |
| taxonomy_id | Int64 | NCBI taxid |
| checksum_sha256 | String | 64 hex chars |

#### atlas_windows_exact.csv
| Field | Type | Constraint |
|-------|------|------------|
| replicon_id | String | FK → atlas_replicons |
| window_length | Int64 | bp |
| window_start | Int64 | 0-indexed, circular |
| orbit_ratio | Float64 | 0.25 ≤ x ≤ 1.0 |
| is_palindrome_R | Bool | |
| is_fixed_RC | Bool | |
| orbit_size | Int64 | ∈ {1, 2, 4} |

#### approx_symmetry_stats.csv
| Field | Type | Constraint |
|-------|------|------------|
| replicon_id | String | |
| window_length | Int64 | |
| d_min | Float64 | ≥ 0 |
| d_min_over_L | Float64 | 0 ≤ x ≤ 1 |
| transform_family | Enum | {dihedral, RC, identity} |

#### dicyclic_lifts.csv
| Field | Type | Constraint |
|-------|------|------------|
| dihedral_order | Int64 | 4, 8, 16 |
| verified_double_cover | Bool | |
| lift_group | String | Dic_n notation |
| relations_satisfied | Bool | |

---

## Implementation Phases

### Phase 1: Foundation
- [ ] Julia `Project.toml` with all dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/agourakis82) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
