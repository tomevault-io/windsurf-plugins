---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working principle: fix it or file it

When a flaw, error, or issue is happened upon — during a review, while implementing something else, while investigating — either **address it right away** if it's in scope and safe to fix, or **lodge it as a GitHub issue as soon as it is confirmed** to be real. Never leave a confirmed finding recorded only in a PR body, a code comment, a chat transcript, or your head: those don't get tracked, and the finding is lost. An issue should carry the file references, a concrete failure scenario, and (where known) a suggested resolution, and cite where it was found.

## Project shape

The Primer is a Socratic AI learning companion for children. The codebase is a **Rust workspace under `src/`** (not the repo root) — every cargo command must be run from `src/`. The workspace targets Rust **edition 2024** and pins toolchain **1.88** in `rust-toolchain.toml` (raised from 1.87 once `primer-gui` pulled in Tauri 2.11, whose transitive deps — darling 0.23, plist 1.9, serde_with 3.20, time 0.3.47 — require 1.88+; the speech vendor's earlier 1.87 floor is satisfied by any 1.88+ build). No system dependencies for default builds: SQLite is bundled, TLS uses rustls. `--speech` mode additionally needs system `espeak-ng`.

**Phase 0.1 is done; Phase 0.2 is done; Phase 0.3 is done.** Hybrid retrieval pipeline, JSONL ingestion + multi-file auto-seed, a 56-passage CC0 hand-drafted seed corpus across all five planned clusters (space, body, how-things-work, life, earth/weather), AND a 35-article Simple English Wikipedia layer (CC-BY-SA-3.0; physics fundamentals, chemistry, biology, earth science, health) all auto-load alongside each other on a fresh KB. Working today: streaming generation against Claude, Ollama, or any OpenAI-compatible server (oMLX, LM Studio, vLLM, etc. via `--backend openai-compat`), SQLite session persistence with resume-by-UUID, long-term memory (rolling LLM summary + FTS5 retrieval), hybrid retrieval (BM25 + dense-vector RRF, default-on; `--embedder-backend` is feature-aware — `fastembed` on a default build, `none` on `--no-default-features`), multi-layer auto-seed corpus, tuned BM25-only AND hybrid retrieval defaults via diagnostic sweeps over the 91-passage corpus (hybrid: 100% loose / 100% strict recall on all 91 queries / 24 strict-subset canonical mappings — `KNOWN_FAILING_QUERIES_HYBRID` is now empty after the issue #45 corpus expansion that added the `seed:en:flowers` passage and a stomach-growl sentence to `seed:en:digestion`), engagement classifier, concept extractor, comprehension classifier, learner-model persistence, per-child vocabulary tracking with Leitner-box spaced repetition, locale-scoped knowledge base, graceful inference-error handling with retry, session-time-based break suggestions, and a working voice round-trip POC (`--speech`). Local llama.cpp inference has landed (`LlamaCppBackend`, in-process GGUF behind the non-default `llamacpp` feature; benchmarking still ahead). An opt-in local→cloud fallback (`FallbackBackend` decorator, `--fallback-backend`/`--fallback-model` CLI flags) has also landed. Still ahead: llama.cpp device benchmarking, voice-loop hardening, hardware integration. See [README.md](README.md) for the user-facing status, [ROADMAP.md](ROADMAP.md) for the phase plan, and [primer_technical_spec.md](primer_technical_spec.md) for the long-form vision.

## Common commands

All commands run from `src/`:

```bash
cargo build                                          # build the workspace
cargo run --bin primer                               # REPL with stub backend (no API key needed)
cargo run --bin primer -- --backend cloud --name Binti --age 8                # uses ANTHROPIC_API_KEY env var
cargo run --bin primer -- --backend cloud --model claude-opus-4-7             # override the default cloud model
cargo run --bin primer -- --backend ollama --model llama3.2                   # local Ollama at http://localhost:11434
cargo run --bin primer -- --backend openai-compat --model mlx-community/Qwen3-8B-4bit   # OpenAI-compatible server (oMLX, LM Studio, vLLM, etc.)
cargo run --bin primer --features qnn -- --backend qnn --qnn-bundle-dir ~/bundles/qwen3-4b   # Qualcomm NPU (Android only at runtime; host emits PlatformUnsupported)
cargo run --bin primer -- --session-db /tmp/explicit.db                       # override default session DB location
cargo run --bin primer -- --resume <uuid>                                     # resume a past session (no greeting)
cargo run --bin primer -- --no-persist                                        # in-memory session, nothing written to disk
cargo test                                           # run all tests (parser + dialogue manager coverage)
cargo test -p primer-pedagogy                        # tests for one crate
cargo test -p primer-pedagogy decide_intent          # single test by name substring
cargo clippy --workspace --all-targets               # lint
cargo fmt                                            # format
RUST_LOG=debug cargo run --bin primer                # verbose tracing output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hherb/primer](https://github.com/hherb/primer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
