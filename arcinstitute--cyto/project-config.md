---
trigger: always_on
description: Core mapping engine. Maps paired-end sequencing reads to features (genes, CRISPR guides) using sequence hashing. Handles barcode correction, UMI extraction, and optional probe demultiplexing. When probes are present, writes per-probe IBU output files; otherwise writes a single IBU. Supports both BINSEQ and FASTX inputs via parallel processing.
---

# cyto-map

## Purpose

Core mapping engine. Maps paired-end sequencing reads to features (genes, CRISPR guides) using sequence hashing. Handles barcode correction, UMI extraction, and optional probe demultiplexing. When probes are present, writes per-probe IBU output files; otherwise writes a single IBU. Supports both BINSEQ and FASTX inputs via parallel processing.

## Key Source Files

- `src/detect.rs` — Geometry detection module. Samples N reads **per input file** independently, scans all positions for known reference sequences using `query_sliding_iter`, infers optimal geometry from position consensus. When multiple files are provided, validates that all files produce the same geometry string and takes the maximum remap window. Types: `DetectionConfig` (`num_reads`, `min_proportion`, `remap_min_proportion`, `num_threads`), `ComponentEvidence` (carries single-position `match_count`/`match_proportion` plus windowed counterparts `windowed_match_count`/`windowed_match_proportion` summed over `[pos ± remap_window]` on the same mate -- approximates what `cyto map --remap-window N` would score), `PerFileResult` (per-file detection summary), `DetectionResult` (includes `per_file_results: Vec<PerFileResult>`). Public API: `detect_gex_geometry()`, `detect_crispr_geometry()` (both consume unpositioned mappers, sample all input files, validate consistency), `log_detection_result()` (logs aggregated result with per-file counts to stderr via `info!`; emits a clearly labeled `Recommended --remap-window: N` line on its own so users can copy it into `cyto map --remap-window N`; the per-component stderr line includes `windowed_count=` and `windowed_proportion=` tokens inline after the existing `count=`/`proportion=` tokens; the bare geometry string itself is printed to stdout by `run_detect_*` in `run.rs`, not by this module). Internal: `PositionAccumulator` (has `windowed_count()` helper that sums hits over `[best_pos.saturating_sub(window), best_pos + window]` inclusive on a given `(component, mate)`), `GexDetectionProcessor`/`CrisprDetectionProcessor` (implement both `binseq::ParallelProcessor` and `paraseq::PairedParallelProcessor`), `infer_geometry()`, `estimate_remap_window()` (proportion-based threshold, contiguous-range walk, includes `[probe]` -- only `[barcode]` is excluded), `resolve_overlaps()`, `validate_and_aggregate()` (takes `Vec<(String, PositionAccumulator, DetectionResult)>`, validates geometry consistency across files, aggregates evidence counts, and re-walks the merged accumulator at `max_remap_window` to populate aggregated windowed counts), `log_per_file_result()`, `log_top_alternatives()`.
- `src/geometry.rs` — Geometry DSL parser and resolver. Parses bracket notation (e.g. `[barcode][umi:12][:10][probe] | [gex]`) into `Geometry` → resolves to `ResolvedGeometry` with concrete byte offsets and read mates. `has_component()` checks whether a geometry includes a given component (used for probe validation). Extensive unit tests (~300 lines).
- `src/mapper/mod.rs` — `Mapper` trait (`query(&self, seq) -> Option<usize>`, `mate() -> ReadMate`), `Library` trait (statistics), typestate markers (`Unpositioned`, `Ready`)
- `src/mapper/gex.rs` — `GexMapper<S>`: maps GEX probes via `SplitSeqHash`. Two-half matching with configurable hamming distance (`GEX_MAX_HDIST=3`). Implements `FeatureWriter`.
- `src/mapper/crispr.rs` — `CrisprMapper<S>`: two-stage matching — `MultiLenSeqHash` for variable-length anchors, then `SeqHash` for fixed-length protospacers. Protospacer offset computed dynamically from anchor match.
- `src/mapper/whitelist.rs` — `WhitelistMapper<S>`: cell barcode correction via `SeqHash`. Returns 2-bit encoded barcodes. Supports parallelized hash build.
- `src/mapper/probe.rs` — `ProbeMapper<S>`: demultiplexing probe mapper with optional regex filtering on aliases. Creates `Bijection` for unique probe-to-index mapping.
- `src/mapper/umi.rs` — `UmiMapper`: extracts UMI from reads, validates quality scores against threshold (`UMI_MIN_QUALITY=10`), provides 2-bit encoding.
- `src/mapper/biject.rs` — `Bijection<T>`: bidirectional map (element <-> index) used for deduplicating probe aliases.
- `src/processor.rs` — `MapProcessor<M>`: parallelized read processing. Handles both probed (multi-output with `Option<ProbeMapper>` and `Option<Bijection>`) and unprobed (single-output) modes in a unified struct. Two constructors: `probed()` and `unprobed()`. Implements both `binseq::ParallelProcessor` and `paraseq::PairedParallelProcessor`. Thread-local buffers flushed on batch complete. Progress bar on thread 0.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ArcInstitute/cyto](https://github.com/ArcInstitute/cyto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
