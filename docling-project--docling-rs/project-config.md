---
trigger: always_on
description: Rust port of [docling](https://github.com/docling-project/docling): document
---

# CLAUDE.md — working notes for AI-assisted sessions

Rust port of [docling](https://github.com/docling-project/docling): document
conversion (PDF/Office/HTML/audio/video/…) to Markdown / docling-JSON / DCLX,
validated for byte-for-byte conformance against upstream Python docling.

## Workflow rules

- **Every commit must be signed off by the author.** End each commit message
  with `Signed-off-by: name <email>`.
- **Releases are automatic on merge; only the `fix:` prefix matters.** CI
  (`scripts/ci/bump_version.sh`): a merge whose commits are all
  `fix:`/`perf:`/`revert:` cuts a patch (0.49.0 → 0.49.1); **any other
  commit** — whatever its prefix — bumps the 0.x "major" (0.49.0 → 0.50.0).
  So prefix bug fixes `fix(scope): …` and don't worry about the rest.
  Docs/CI-only merges still release nothing (release.sh only fires when a
  publishable crate's source changed). No automatic semver-major: majors are
  cut by hand (`force_version` dispatch input) — v1.0.0 marks the breaking
  `.models/` asset-dir rename.
- Claude Web: **Never open pull requests on `artiz/docling.rs`.** Push a `claude/<topic>`
  branch and hand back a compare link
  (`https://github.com/docling-project/docling.rs/compare/master...artiz:docling.rs:<branch>?expand=1`);
  the maintainer opens/merges PRs themself (usually into the upstream
  `docling-project/docling.rs`; `artiz/docling.rs` is their working fork).
- One feature = one branch off fresh `origin/master`. Don't stack unrelated
  work.
- Issue numbers (`#80`, `#138`, …) refer to `docling-project/docling.rs`
  issues; reference them in commit messages (`Refs #NN`).
- **Slack notifications** (when the Slack MCP connector is available): post to
  **#claude-code** (channel ID `C0BKAHN0BSM`) when (a) a question blocks the
  work and needs the maintainer's answer, (b) a long task finishes — include
  the outcome and the compare link, (c) idle/bored — the queue is empty;
  suggest what to pick up next. Keep it to these events; don't narrate
  routine progress there.

## Workspace map

| Crate | What it is |
|---|---|
| `crates/docling-core` | `DoclingDocument` model, Markdown/JSON/DCLX serializers, `MarkdownStreamer`, chunkers |
| `crates/docling` | `DocumentConverter` (format routing), declarative backends (`src/backend/`), streaming (`src/stream.rs`), video (`src/video.rs`) |
| `crates/docling-pdf` | ML pipeline: pdfium + RT-DETR layout + TableFormer + PP-OCRv3 + enrichment (`ml` feature); pure-Rust text-layer path compiles for wasm without it |
| `crates/docling-asr` | Whisper ASR: symphonia decode (audio + video containers) → log-mel → ONNX encoder/decoder |
| `crates/docling-cli` | `docling-rs` binary (also `serve` subcommand behind `--features serve`) |
| `crates/docling-serve` | axum HTTP conversion API (+ Dockerfile with ffmpeg) |
| `crates/docling-py` / `docling-node` / `docling-wasm` | pyo3 / napi-rs / wasm-bindgen bindings — **node and wasm are ordinary workspace members; only docling-py sits outside the workspace and builds from its own directory (maturin)** |
| `crates/docling-rag` | RAG subsystem (embedder, store, web UI) |

## Build & test

```bash
cargo test --lib --tests -p docling-core -p docling -p docling-asr -p docling-serve -p docling-pdf
cargo clippy --lib --tests --bins <same -p list>   # keep it warning-free
cargo fmt --all
cargo check -p docling --no-default-features --features pdf-text \
  --target wasm32-unknown-unknown --locked           # the wasm CI gate
(cd crates/docling-py && cargo check)               # pyo3 binding (outside the workspace)
```

- Prefer `--lib --tests` over bare `cargo test`: it skips example binaries,
  each of which statically links onnxruntime (~0.3–5 GB of `target/` churn).
- **Disk discipline (remote container!):** `target/debug` balloons past 15 GB.
  When "No space left on device" hits, delete `target/debug/examples`,
  `target/debug/incremental`, oldest `target/debug/deps` files — deletes work
  even at 0 free. `CARGO_INCREMENTAL=0` helps. Old rustup toolchains and
  `~/.cargo/registry/cache` are also safe to drop.
- Tests run with CWD = the crate dir, but shared fixtures and runtime assets
  live at the **repo root**. Resolve fixtures via
  `Path::new(env!("CARGO_MANIFEST_DIR")).join("../..")` and gate asset-needing
  tests with a skip (see `crates/docling/tests/pages.rs::pdfium_ready`,
  `crates/docling/src/video.rs::asr_models_ready`) — CI without models/pdfium
  must stay green.

## Runtime assets & env

- `.models/` (repo root): layout, TableFormer, OCR, ASR (`.models/asr/`,
  presets in subdirs), enrichment, embedder. `.pdfium/lib/libpdfium.so` for
  page rendering. Fetch: `scripts/install/download_dependencies.sh`.
- Resolution is CWD-relative with exe-dir fallback; env overrides:
  `PDFIUM_DYNAMIC_LIB_PATH`, `DOCLING_ASR_{ENCODER,DECODER,VOCAB}`,
  `DOCLING_FFMPEG` (video frames — ffmpeg is a runtime binary, never a build
  dep), `DOCLING_RS_PDF_WORKERS/_THREADS/_INTRA`, `DOCLING_RS_TF_INTRA` (#262),
  `DOCLING_RS_NO_ARENA` (#263; serve defaults it on),
  `DOCLING_RS_MAX_MEMORY_MB` + `DOCLING_RS_MEMORY_WATERMARK_PCT` (serve
  admission control), `DOCLING_RS_FP32`,
  `DOCLING_RS_EP` (GPU execution providers), `DOCLING_RS_ASR_LANG`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [docling-project/docling.rs](https://github.com/docling-project/docling.rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
