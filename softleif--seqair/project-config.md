---
trigger: always_on
description: Pure-Rust BAM/SAM/CRAM/FASTA reader + pileup engine. I/O backend for [rastair](https://github.com/bsblabludwig/rastair).
---

# seqair

Pure-Rust BAM/SAM/CRAM/FASTA reader + pileup engine. I/O backend for [rastair](https://github.com/bsblabludwig/rastair).

Workspace: `crates/seqair` (readers, pileup, BGZF, CRAM) and `crates/seqair-types` (Base, Strand, Phred, Probability, RmsAccumulator, RegionString).

## Tracey specs

Specs in `docs/spec/*.md` with `r[rule.id]`. Code: `// r[impl rule.id]`, tests: `// r[verify rule.id]`. Use the `tracey` skill. Add spec rules before implementing; update specs when changing behavior.

## Coding style

Expert Rust. Modern idioms. Types are the primary abstraction.

- Correctness and clarity first. Comments explain "why" only.
- No `mod.rs` — use `src/some_module.rs`.
- No `unwrap()` — propagate with `?`.
- No indexing — use `.get()`. If indexing is unavoidable, `#[allow(clippy::indexing_slicing)]` + `debug_assert!`.
- No `let _ =` on fallible ops — propagate, log with `warn!`, or handle.
- No `from_utf8_lossy` — use `from_utf8()?` with typed errors.
- Error enums: one per module, typed fields only (never `String`), `#[from]` for wrapping. Never use `io::Error::other("message")` — add a typed variant instead. Hierarchy: `BgzfError` → `BamHeaderError`/`BaiError` → `BamError`; `BamWriteError` (parallel to `BamError` for the write path); `FaiError`/`GziError` → `FastaError`; `FormatDetectionError` → `ReaderError`; `VcfHeaderError`/`VcfEncodeError`/`AllelesError` → `VcfError`.
- `color_eyre` for errors, `tracing` for logging.
- Sequence names are `SmolStr`.
- Tests: prefer `cargo nextest run` (parallel test-binary execution; ~8× faster than `cargo test` on this repo, config in `.config/nextest.toml`). Doctests aren't run by nextest — append `cargo test --doc --quiet` when you need to cover them. Prefer `proptest` where applicable. Round-trip tests against noodles and bcftools are the strongest validation — always add them for new output formats. Avoid tautological tests — don't verify code using a copy of the same logic (e.g., compute expected bin with the same `reg2bin`). Use independent oracles. For tests that write temp files (e.g., bcftools round-trips), use the `tempfile` crate (`tempfile::NamedTempFile`) rather than `std::env::temp_dir().join(...)` to avoid filename collisions under parallel test execution.
- `SmallVec` in this project uses 2-arg form `SmallVec<T, N>` (not `SmallVec<[T; N]>`).
- No silent `as i32`/`as u32` truncation at serialization boundaries — use `i32::try_from()` or equivalent with typed errors. At allocation boundaries (parsing untrusted counts), apply practical upper bounds matching real-world data, not format maximums (`i32::MAX` is not a limit). See `r[io.writer_limits]` and `r[io.fuzz.alloc_limits]` in the general spec.

## Pre-push CI parity

CI runs clippy and tests with `-D warnings`. **Before pushing, always run with the same flag:**

Frequent CI-only failures to watch for: `clippy::cast_possible_truncation`, `clippy::arithmetic_side_effects`, `clippy::doc_markdown`, `clippy::field_reassign_with_default`, `clippy::needless_update`, `clippy::useless_conversion`, `clippy::unnecessary_cast`

## Architecture notes

**RecordStore**: 4 contiguous Vecs (records, names, bases, data). Zero per-record heap alloc.

**RegionBuf**: bulk-reads compressed bytes for a region, decompresses from memory. Uses `Vec<RangeMapping>` for disjoint chunks — never subtract offsets directly.

**ChunkCache**: `BamIndex::query_split()` separates nearby (L3–L5) from distant (L0–L2) BAI chunks. Distant chunks loaded once per tid per thread.

**CigarMapping**: `Linear` fast-path for clip+match (~90%), `Complex` with `SmallVec<6>`. Pre-extracted at construction.

**PileupAlignment**: base/qual/mapq/flags/strand pre-extracted. Hot loop reads flat fields only.

**PileupOp enum**: type-safe indel reporting — `Match`/`Insertion` carry `qpos`/`base`/`qual`, `Deletion` carries only `del_len`, `ComplexIndel` carries `del_len`+`insert_len` (deletion with following insertion, e.g. `D I M` in CIGAR), `RefSkip` carries nothing. Compiler prevents reading a base from a deletion. Deletions and ref-skips are included in columns (not filtered out). `depth()` counts all alignments (matches htslib); `match_depth()` counts only those with a query base. Insertions attach to the last M/=/X position before the I op; `D I M` patterns emit `ComplexIndel` at the last D position (matching htslib's `is_del=true, indel>0`). `del_len` is the total D op length at every position within the deletion (not remaining bases). `PileupOp` has a compile-time size guard (≤16 bytes).

**FASTA**: returns raw `Vec<u8>` (not `Vec<Base>`) — CRAM MD5 needs exact bytes. Conversion to `Base` at app boundary.

**Base::known_index()**: A/C/G/T → `Some(0..3)`, Unknown → `None`. Zero-depth pileups and Unknown ref bases are valid states.

**Forkable readers**: `Arc<BamShared>` (index + header) parsed once; `fork()` gives fresh File handle + ChunkCache.

**CRAM**: v3.0/v3.1. Multi-ref slices (ref_seq_id == -2), span=0 CRAI entries included in queries, embedded references, coordinate clamping to i64::MAX, rANS order-1 chunk-based interleaving, per-slice MD5 verification.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Softleif/seqair](https://github.com/Softleif/seqair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
