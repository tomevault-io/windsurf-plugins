---
trigger: always_on
description: operating notes for ai agents and human contributors working in this repo. the principal author writes fast and uses voice transcription. typos, caps lock, missing accents are common. read intent, do not flag tone, do not project emotional risk.
---

# user guide

operating notes for ai agents and human contributors working in this repo. the principal author writes fast and uses voice transcription. typos, caps lock, missing accents are common. read intent, do not flag tone, do not project emotional risk.

# build and test

- `cargo build --workspace` / `cargo build --release --workspace`
- `cargo test --workspace`: all rust tests (unit + integration + golden), 288/288 on the current unreleased state (`forge-core` adds 6 more on its own manifest)
- `cargo fmt --all --check`: formatting check
- `cargo clippy --workspace --all-targets -- -D warnings`: linting (warnings are errors)
- `ruff check .` / `ruff format --check .`: python linting and formatting (config in `pyproject.toml`)
- `./scripts/release_check.sh`: full pipeline + regression gates against `dat/measure/baseline.json`. single source of truth for "PR-ready". exits non-zero on any failure.
- `forge-core` (the ingestion layer) is a SEPARATE cargo workspace OUTSIDE `crates/`; the sovereign `--workspace` commands and `release_check.sh` do not touch it. build and test it on its own manifest: `cargo build --manifest-path forge-core/Cargo.toml`, `cargo test --manifest-path forge-core/Cargo.toml`, `cargo clippy --manifest-path forge-core/Cargo.toml --all-targets -- -D warnings`, `cargo fmt --manifest-path forge-core/Cargo.toml --all --check`. the <=300-line rule applies there too (release_check's guard only scans `crates/`).

# pyo3 extension

no maturin. build manually:

```
cargo build --release -p nest-python
cp target/release/lib_nest.dylib python/_nest.so   # macOS
cp target/release/lib_nest.so   python/_nest.so    # linux
```

abi3 targets python 3.12+ (not 3.14). python tests need the built `.so` first:

```
python tests/test_e2e.py
python tests/test_builder.py
python tests/test_search_text_model_hash.py
```

no pytest. tests are plain scripts with `if __name__ == "__main__"`. `pytest tests/` does not work.

the forge build-side default embedder is now the REAL SEMANTIC one: a vendored model2vec/potion-base-8M static table (`python/forge/embed_potion.py`), offline, no torch, no network. its self-test needs numpy + tokenizers and the vendored table (git-lfs): `python python/forge/test_embed_potion.py` (run with `.venv/bin/python`; install the deps with `uv pip install numpy tokenizers`, declared in `pyproject.toml` under `[dependency-groups] forge`). it proves the semantic jump (car ~ automobile >> car ~ banana), determinism, f32-stability, and that no socket is opened at embed time; `python python/forge/recall_harness.py` shows per-query recall vs the floor. the table (`python/forge/models/potion-base-8M/model.safetensors`, ~30mb) is git-lfs; run `git lfs pull` if it is a pointer. the #04 lexical bag-of-words stays as the stdlib-only zero-dep FLOOR with its own self-test (no `.so`, no deps): `python python/forge/test_embed_default.py`. both self-fingerprint to a `model_hash` recorded in provenance; neither is run by `release_check.sh`.

# single-target commands

- `cargo test -p nest-format`: format crate only
- `cargo test -p nest-runtime`: runtime crate only
- `cargo test --release -p nest-runtime --test hnsw_recall`: HNSW recall regression (needs release; debug is too slow to run within timeout)
- `cargo test -p nest-cli`: CLI integration tests (requires release build)
- `cargo run -p nest-format --example regen_golden`: regenerate the byte-frozen golden fixture

# architecture

```
nest-format  standalone library (binary format spec, reader, writer, manifest, encoding, hashing)
nest-runtime depends on nest-format (mmap-backed search, MmapNestFile, ann::HnswIndex, bm25::Bm25Index, graph::CsrIndex, simd dispatcher)
nest-cli     depends on nest-format + nest-runtime (clap binary, 9 engine subcommands + the ask/retrieve flagship verbs)
nest-python  depends on nest-format + nest-runtime (cdylib _nest, PyO3 abi3-py312)

forge-core   SEPARATE cargo workspace at the repo root, OUTSIDE crates/ (ingestion layer,
             FORGE-0a: the frozen .fci canonical-intermediate schema). its deps never enter the
             sovereign crates; not in the `--workspace` set. .fci is versioned independently.
```

CLI binary: `nest`. nine engine subcommands: `inspect`, `validate`, `search`, `search-ann`, `search-graph`, `search-text`, `benchmark`, `stats`, `cite`. plus two agent-native flagship verbs layered over the same engine: `ask` (text query in, cited answer out, `--disclose answer|explain`) and `retrieve` (json/jsonl answer-pack of cited spans where score IS the exact rerank value). the flagship keeps the nine subcommands as-is under the hood; verb-collapse, the `nest dev` namespace, and the nest-profile crate are deferred (churn with no user value pre-users).

python entry: `sys.path.insert(0, "python"); import nest`. dynamic loader finds `_nest.so` or `lib_nest.dylib`.

# repo workflow

- remote: `git@github.com:hoffresearch/nest.git`. owner: hoff research. maintainer: brenner cruvinel (`brenner@hoffresearch.com`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hoffresearch/nest](https://github.com/hoffresearch/nest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
