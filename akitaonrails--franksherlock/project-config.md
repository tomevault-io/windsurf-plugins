---
trigger: always_on
description: Frank Sherlock is a local-only, AI-powered image cataloging and search system. The main application lives in `sherlock/` — a Tauri v2 desktop app with a Rust backend and React frontend. Everything else in this repo is research and prototyping that informed the final design.
---

# CLAUDE.md

## Project Overview

Frank Sherlock is a local-only, AI-powered image cataloging and search system. The main application lives in `sherlock/` — a Tauri v2 desktop app with a Rust backend and React frontend. Everything else in this repo is research and prototyping that informed the final design.

## Repository Layout

```
sherlock/               <- THE MAIN APP (Tauri v2 desktop)
  desktop/
    src-tauri/          <- Rust backend (scan, classify, thumbnail, db, config)
    src/                <- React frontend (VSCode-inspired UI)
_classification/        <- PoC: Python classification pipeline (informed classify.rs)
_research_ab_test/      <- A/B benchmark research (scripts, docs, test files)
  scripts/              <- Benchmark scripts for model selection
  docs/                 <- Research notes: IDEA.md, IDEA2.md, RESULTS.md
  lib/                  <- Shared Python helpers
  test_files/           <- Test corpus (gitignored, not shipped)
  results/              <- Generated benchmark outputs (gitignored)
scripts/                <- Build scripts (build-local.sh)
docs/                   <- App assets (app_icon.png)
.github/workflows/      <- CI/CD (release.yml, ci.yml)
```

## Tech Stack (Main App)

- **Desktop**: Tauri v2.10+ (Rust + WebView)
- **Backend**: Rust (rusqlite, ureq, image, base64, regex, sha2, walkdir)
- **Frontend**: React + Vite + TypeScript
- **AI**: Ollama (qwen2.5vl:7b) for vision classification, Surya OCR in isolated Python venv
- **Database**: SQLite + FTS5 in `~/.local/share/frank_sherlock/db/`
- **Target OS**: Linux (AppImage), macOS (DMG), Windows (MSI)

## Build & Test

From `sherlock/desktop/`:

```bash
npm install                    # frontend deps
cargo build                    # rust backend (from src-tauri/)
cargo test                     # 322 unit tests
npm run tauri:dev              # launch dev mode
npm run test                   # 299 frontend tests
npm run tauri:build            # produce AppImage/DMG/MSI
```

If Wayland/NVIDIA causes WebKit issues:
```bash
WEBKIT_DISABLE_DMABUF_RENDERER=1 GDK_BACKEND=wayland,x11 npm run tauri:dev
```

## Key Rust Modules

| Module | Purpose |
|--------|---------|
| `classify.rs` | Ollama vision LLM pipeline: primary classification (3-attempt + regex salvage), anime enrichment, Surya OCR + LLM fallback, document/receipt extraction |
| `face.rs` | Native ONNX face detection (SCRFD) and recognition (ArcFace), 512-dim embeddings, cosine clustering, person management |
| `thumbnail.rs` | 300px JPEG thumbnails, Lanczos3, skip-if-exists, GIF first-frame |
| `scan.rs` | Two-phase incremental scan: metadata-only discovery (zero reads for unchanged), classification + thumbnail processing, move detection, cache cleanup, cooperative cancellation |
| `db.rs` | SQLite + FTS5, scan job checkpointing, upsert/move/delete operations |
| `config.rs` | AppPaths resolution, directory creation |
| `lib.rs` | Tauri commands, scan worker spawning, setup/download flow, auto-cleanup, scan cancellation |
| `query_parser.rs` | Natural language query parsing (media type, dates, confidence) |
| `runtime.rs` | Ollama/nvidia-smi status gathering |
| `platform/` | OS abstraction: clipboard, GPU detection, Python venv paths, executable lookup |

## Architecture Principles

- **Read-only**: Never writes to scanned directories. All data goes to `~/.local/share/frank_sherlock/`.
- **Incremental**: Unchanged files (same mtime + size) skip fingerprinting and classification entirely — only a scan marker update.
- **Move-aware**: Renamed/moved files detected by fingerprint, preserving all classification data.
- **Resilient**: Scan jobs checkpoint after each file. Interrupted scans resume from last cursor.
- **Local-only**: No cloud APIs. Ollama runs locally, Surya OCR runs in an isolated venv.
- **Multi-OS**: All features must work on Linux, macOS, and Windows. OS-specific code lives exclusively in `src-tauri/src/platform/`. Never use OS-specific paths, commands, or assumptions outside this module. CI tests all three platforms on every push.

## Coding Conventions

- Rust: use `thiserror` for error types via `AppError` enum. Propagate with `?`, never `.unwrap()` in production code.
- All new Rust modules must have `#[cfg(test)] mod tests` with unit tests.
- Frontend: TypeScript strict mode, camelCase for TS types matching Rust's `#[serde(rename_all = "camelCase")]`.
- JSON parsing from LLMs uses a 3-tier fallback: direct parse -> brace-balanced extraction -> regex field salvage.
- Thumbnails and classification caches mirror the source `rel_path` structure under their respective dirs.
- Frontend shared utilities (`basename`, `errorMessage`) live in `src/utils.ts`. Modal CSS shares base styles via `shared-modal.css`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akitaonrails/FrankSherlock](https://github.com/akitaonrails/FrankSherlock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
