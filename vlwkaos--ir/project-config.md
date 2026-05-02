---
trigger: always_on
description: Package name on crates.io is `ir-search` (name `ir` was taken).
---

# ir — Agent Instructions

## Crate

Package name on crates.io is `ir-search` (name `ir` was taken).
Binary name is `ir`. See @Cargo.toml.

## Commands

```bash
cargo build                        # dev build
cargo build --release --bin ir     # release build
cargo test                         # unit tests (fast, no models needed)
cargo test -- --ignored            # includes LLM tests (require model files)
```

Benchmark runner (drives the real `ir` binary; requires BEIR dataset):
```bash
scripts/bench.sh fiqa              # bench current HEAD on FiQA
scripts/bench.sh fiqa v0.9.0       # compare HEAD vs v0.9.0
scripts/bench.sh miracl-ko         # Korean MIRACL benchmark
```
Results cached at `logs/results/{dataset}/{git7}.json` (gitignored).

## Environment Variables

| Var | Default | Description |
|-----|---------|-------------|
| `IR_EMBEDDING_MODEL` | auto-detect | Path to embedding GGUF |
| `IR_EXPANDER_MODEL` | auto-detect | Path to expander GGUF (qmd-1.7B) |
| `IR_RERANKER_MODEL` | auto-detect | Path to reranker GGUF (Qwen3-0.6B) |
| `IR_COMBINED_MODEL` | unset | Unified Qwen3.5 GGUF — replaces both expander + reranker (`IR_QWEN_MODEL` deprecated alias) |
| `IR_GPU_LAYERS` | `99` on macOS | Number of layers offloaded to GPU |
| `IR_FORCE_CPU_BACKEND` | unset | Set to `1` to disable Metal |
| `IR_LLAMA_LOGS` | unset | Set to `1` to enable llama.cpp verbose logging |
| `IR_MODEL_DIRS` | `~/local-models/` | Colon-separated extra model search dirs |
| `IR_CONFIG_DIR` | `~/.config/ir` | Override config/data dir. Supports `~` and `$VAR` expansion. |
| `XDG_CONFIG_HOME` | `~/.config` | **Deprecated** — use `IR_CONFIG_DIR` instead. Still works but emits a warning. |
| `IR_BENCH_SIGNALS` | unset | Research: emit `SIGNAL_FUSED\ttop\tgap` to pipeline log for threshold tuning |
| `IR_DISABLE_SHORTCUTS` | unset | Research: disable BM25 + fused strong-signal shortcuts for A/B benchmarking |
| `IR_FORCE_TIER1_ONLY` | unset | Research: force hybrid to return tier-1 fused results only (skip tier-2) |
| `IR_STRONG_SIGNAL_FLOOR_OVERRIDE` | unset | Research: override fused strong-signal floor threshold |
| `IR_STRONG_SIGNAL_PRODUCT_OVERRIDE` | unset | Research: override fused strong-signal product threshold |
| `IR_STRONG_SIGNAL_PRODUCT_PREPROCESSED_OVERRIDE` | unset | Research: override fused strong-signal product for preprocessed (Korean) collections |
| `IR_BM25_STRONG_FLOOR_OVERRIDE` | unset | Research: override BM25 strong-signal floor threshold |
| `IR_BM25_STRONG_GAP_OVERRIDE` | unset | Research: override BM25 strong-signal gap threshold |
| `IR_ALLOW_EXPANSION_WITHOUT_SCORER` | unset | Research: allow expansion without reranker (harmful in production: -0.53% nDCG on NFCorpus) |

Config dir precedence: `IR_CONFIG_DIR` → `XDG_CONFIG_HOME/ir` (deprecated) → `~/.config/ir`

Model search order: `IR_*_MODEL` env → `IR_MODEL_DIRS` → `~/local-models/` → `~/.cache/ir/models/` → `~/.cache/qmd/models/`

`QMD_EMBEDDING_MODEL`, `QMD_EXPANDER_MODEL`, `QMD_RERANKER_MODEL` are also checked as fallbacks.

All path env vars (`IR_CONFIG_DIR`, `IR_MODEL_DIRS`, `IR_*_MODEL`) support `~` and `$VAR`/`${VAR}` expansion.

Note: `IR_DIR` is set internally at startup (= resolved `ir_dir()` value). It appears in preprocessor commands stored in `config.yml` as `$IR_DIR/preprocessors/...` so they are portable. Not user-facing.

## Data Paths

- Config: `~/.config/ir/config.yml`
- Collection DBs: `~/.config/ir/collections/{name}.sqlite`
- Expander cache: `~/.config/ir/expander_cache.sqlite`
- Daemon socket: `~/.config/ir/daemon.sock`

## Architecture

### Search Pipeline

Three-tier escalation. Each tier runs only if the previous tier's result isn't strong enough.

| Tier | Models | Enables |
|------|--------|---------|
| 0 | none | BM25 (FTS5), in-process |
| 1 | Embedder | Vector, hybrid score-fusion (0.80·vec + 0.20·bm25) |
| 2 | Expander + Scorer | Query expansion (lex/vec/hyde → RRF) + reranking |

Strong-signal shortcut: raw BM25 top ≥ 0.75 AND gap ≥ 0.10 → skip Tier 1+2 entirely (`src/search/hybrid.rs:is_bm25_strong_signal`). Expander without scorer is a no-op (`hybrid.rs:112`).

See @research/pipeline.md for diagrams.

### Daemon Startup

Staged async: BM25 runs in-process immediately. Daemon starts in background.

- Tier 1 ready: embedder loaded → socket bound → client unblocks (waits up to 3s)
- Tier 2 ready: expander + reranker loaded → tier2 signal file written → client re-queries if needed (waits up to 7s)

Idle timeout: 3600s (configurable via `ir daemon start --timeout`).

## Known Gotchas

- **LLM tests are `#[ignore]`**: `cargo test` skips them. Run `cargo test -- --ignored` only when model files are present.
- **sqlite-vec must be registered before any connection opens**: `ensure_sqlite_vec()` uses `sqlite3_auto_extension` (process-global). Called once via `OnceLock` in `db/mod.rs`.
- **`LlamaBackend` is a singleton**: `OnceLock<LlamaBackend>` in `src/llm/mod.rs`. Loading a second model in the same process does NOT call `init()` again — this is intentional.
- **Daemon requires restart after binary change**: `ir search` auto-starts the daemon but won't restart a running one. Kill it with `ir daemon stop` after rebuilding.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vlwkaos/ir](https://github.com/vlwkaos/ir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
