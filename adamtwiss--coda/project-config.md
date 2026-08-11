---
trigger: always_on
description: Coda is a UCI chess engine in Rust, rewritten from GoChess.
---

# CLAUDE.md — Coda Chess Engine

Coda is a UCI chess engine in Rust, rewritten from GoChess.
**Chess Optimised, Developed Agentically** — built through human-AI collaboration.

> Coda's development methodology (testing, SPRT/SPSA, OpenBench), NNUE training,
> cross-engine research, tooling, and the Claude skills live in a **separate
> private research repository**. This public repo is the engine: source, build,
> tests, and the licensing docs (`docs/license_analysis_2026-07-13.md`).
>
> **Never add scripts or tuning artefacts to this repo.** There is no `scripts/`
> directory and there should not be one — it is `.gitignore`d, along with
> `tune_*.txt`. Research tooling, launchers, analysis scripts and SPSA specs all
> belong in the private repo. If a tool seems to need to live here, it is almost
> certainly a `coda` subcommand instead (`bench`, `perft`, `epd`, `tune-spec`,
> `fetch-net`), which is how this project ships tooling. **SPSA specs especially
> must never be committed**: they drift from the `tunables!` macro as soon as a
> tune is applied, and `coda tune-spec` emits the live values on demand, so a
> checked-in copy is guaranteed to be the stale one.

## Supported CPU families

- **x86_64 (primary):** default target.
- **aarch64 (first-class, since 2026-04-25):** Apple M-series, ARM servers. New
  SMP code must use correct memory ordering — `Acquire/Release` on shared atomics
  with reader-publish patterns, not `Relaxed` (x86's strong model masks ordering
  bugs that fire on ARM). Default to `Acquire/Release` + explicit NEON tests.

## Build and Test

Prerequisites: Rust 1.70+. PGO builds also need `cargo install cargo-pgo` +
`rustup component add llvm-tools-preview`.

```bash
make                  # Build with embedded NNUE net + native CPU (produces ./coda)
make net              # Download production NNUE net (from net.txt)
cargo build --release # Plain release (no embedded net)
cargo test            # Run all tests including perft

./coda bench [depth]                  # Search benchmark, 48 positions @ default depth 12
./coda                                # UCI mode
./coda --nnue net.nnue                # UCI with explicit NNUE (-n is the short form; -nnue single-dash is INVALID)
./coda --nnue net.nnue --book book.bin  # ... + opening book
./coda epd wac.epd --nnue net.nnue -t 1000    # epd: -t <ms/pos>, -m <max>, -n/--nnue <net>
./coda perft [depth] [fen...]
./coda perft-bench                    # 6-position perft suite
./coda convert-bullet [options]       # quantised.bin → .nnue
./coda fetch-net                      # Pull net from net.txt URL
./coda help
```

## Project Structure

```
src/
  main.rs          Entry point, CLI argument parsing, subcommands
  board.rs         Board struct (bitboards + mailbox), FEN, make/unmake, Zobrist
  types.rs         Color, Piece, Square, Move encoding (16-bit), castling
  bitboard.rs      Bitboard ops, between/line tables
  attacks.rs       Magic bitboards (PEXT runtime detected), knight/king/pawn tables
  setwise.rs       Setwise (batched) attack generation — all pieces of one type at once
  movegen.rs       Pseudo-legal + capture-only move generation, perft
  zobrist.rs       Zobrist hash keys (deterministic PRNG)
  zobrist_keys.rs  Auto-generated Zobrist key constants
  eval.rs          PeSTO material+PST eval (fallback), SEE values, NNUE eval wrapper
  see.rs           Static Exchange Evaluation
  tt.rs            Transposition table (5-slot buckets, XOR key verification)
  movepicker.rs    Staged move ordering, 4D history tables, continuation history
  search.rs        Negamax, pruning, LMR, correction history, cuckoo, pruning stats
  thread_pool.rs   Persistent Lazy-SMP helper thread pool (reused across go commands)
  cuckoo.rs        Cuckoo cycle detection for proactive repetition avoidance
  tb.rs            Syzygy tablebase probing (via shakmaty-syzygy)
  tb_cache.rs      Lockless Zobrist-keyed WDL probe cache (UCI TBHash)
  nnue.rs          NNUE v5/v7/v9 inference, accumulator stack, Finny table, AVX2/AVX-512/VNNI SIMD
  nnue_simd.rs     NNUE SIMD primitive abstractions (cfg(target_feature)-gated)
  sparse_l1.rs     Sparse/dense int8 L1 matmul kernels (AVX2, AVX-VNNI, AVX-512 VNNI)
  threats.rs       Threat-feature enumeration + delta generation (v9)
  threat_accum.rs  Per-ply threat accumulator stack (v9)
  uci.rs           UCI protocol (position, go, stop, ponder, setoption)
  epd.rs           EPD test suite runner with SAN formatting
  book.rs          Polyglot opening book support
  polyglot_randoms.rs  Standard Polyglot Zobrist random table (781 entries)
  datagen.rs       Multi-threaded training data gen; writes SF BINP binpack via the sfbinpack crate
  bullet_convert.rs  Bullet quantised.bin → .nnue converter (v5/v7/v9)
  nnue_export.rs   .nnue → Bullet checkpoint converter (for transfer learning)
Makefile           Build targets: make, make pgo, make openbench, make net
net.txt            Production NNUE net URL (used by make net / fetch-net)
```

## Architecture

### Board
Bitboards (`pieces[6]` by type + `colors[2]`) + mailbox (`[u8;64]` for O(1)
piece-at-square). Magic bitboards for sliders (PEXT on BMI2, runtime-detected).
Incremental Zobrist + pawn hash.

### Move encoding

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adamtwiss/coda](https://github.com/adamtwiss/coda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
