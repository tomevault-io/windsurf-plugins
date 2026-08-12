---
trigger: always_on
description: `mm` provides fast, multimodal context for **agents** on the CLI, designed to provide multimodal understanding for file types that are not natively understood by LLMs — images, video, audio, PDFs, and other binary/media formats.
---

# CLAUDE.md — mm

## What this is

`mm` provides fast, multimodal context for **agents** on the CLI, designed to provide multimodal understanding for file types that are not natively understood by LLMs — images, video, audio, PDFs, and other binary/media formats.

Rust core for speed, Python for developer experience, Unix philosophy for composability.

## Core ideology

- Unix philosophy for composability.
- Speed, compression, devex is all what matters.
- Rust core for speed + Python for developer experience.
- Information-theoretical perspective on context
    - Input tok/img or tok/px: PDF/image content measured in toks (tok), dimensions in pixels (px).
    - Input tok/s: audio/video content measured in toks (tok), duration in seconds (s)
    - Input tok/MB: audio/video content, Mtok/MB.

## Best practices

### Testing and benchmarks

- Every performance-oriented or performance-critical method **must** have both unit tests and benchmark coverage. No exceptions — if it's on the hot path, prove it with numbers.
- Write Criterion benchmarks for Rust (in `crates/mm-core/benches/`) and pytest-benchmark tests for Python (in `tests/python/test_benchmark.py`). When adding a new method that touches I/O, parsing, hashing, or serialization, add a corresponding benchmark in the same PR.
- Benchmarks are not afterthoughts. Treat them as first-class artifacts — they catch regressions that unit tests cannot.

### Rust-first for performance

- Default to implementing performance-critical logic in Rust and exposing it to Python via PyO3 bindings. The pattern: Rust core method → PyO3 wrapper in `mm-python` → Python type stub in `_mm.pyi` → importable from `mm`.
- If a Python method shows up in profiling or processes data at scale (file I/O, hashing, parsing, batch transforms), it is a candidate for Rust. Prototype in Python if needed, but graduate to Rust before shipping.
- Keep the Rust → Python boundary clean: pass Arrow IPC bytes, primitive types, or simple structs. Avoid complex Python objects crossing the FFI.

### Code style — Python

- **No header/separator comments.** Do not write `# === Section ===` or `# --- helpers ---` dividers. The code structure should speak for itself.
- **Google-style docstrings** for all public functions, classes, and modules. Document args, returns, raises, and include a short usage example for non-obvious APIs. Internal helpers get a one-liner docstring if the name isn't self-explanatory.
- **Zen of Python applies.** Beautiful is better than ugly. Simple is better than complex. Flat is better than nested. Readability counts. Write code that a staff or principal engineer would be proud to review — elegant, minimal, and intentional. No sloppy shortcuts, no over-engineering.
- Prioritize developer experience and performance equally. Neither is negotiable. If a design forces a tradeoff between the two, find a third option.
- Use type annotations on all public APIs. Leverage `typing` and `typing_extensions` to make interfaces self-documenting.
- **Think in abstractions.** Prefer classes with `@classmethod` / `@staticmethod` and `abc.ABC` / metaclasses over loose collections of functions. Design at the right abstraction level — group related behavior into cohesive classes, define clear interfaces via abstract base classes, and use the type system to enforce contracts. Flat function-based code is fine for scripts; library code should express intent through well-designed object hierarchies.

### Code style — Rust

- Follow idiomatic Rust: use `clippy::pedantic` as guidance, prefer zero-copy where possible, and keep allocations off the hot path.
- Public APIs get `///` doc comments with examples. Internal functions get `//` comments only where the intent isn't obvious from the code.

## Libraries

**Python (core):**
- openai — chat/completions SDK (any OpenAI-compatible API: Ollama, vLLM, OpenAI)
- typer — CLI framework
- rich — terminal formatting (tables, panels, trees, syntax highlighting)
- polars — zero-copy DataFrame from Arrow
- pandas — DataFrame export
- sqlite-vec — SQLite + vector search (global DB at ~/.local/share/mm/mm.db)
- pyarrow — Arrow IPC deserialization (Rust → Python data transfer)
- pypdfium2 — PDF text extraction and page rendering
- Pillow — image mosaic tiling
- tomli — TOML config parsing (Python <3.11)
- pyyaml — YAML template parsing
- faster-whisper — Whisper transcription (CTranslate2 backend)
- scenedetect — shot/scene boundary detection (opencv-python bundled since 0.7)
- ctranslate2 — CTranslate2 inference runtime (for faster-whisper)
- libreoffice-rs — Office Docs text extraction

**Python (mm[mlx]):**
- mlx — Apple Metal GPU acceleration
- lightning-whisper-mlx — MLX Whisper backend (~3-4x faster on Apple Silicon)

**Python (mm[experimental]):**
- datasets — HuggingFace Datasets export

**Rust (mm-core):**
- arrow / parquet — Arrow RecordBatch + Parquet I/O
- pyo3 — Python bindings
- rayon — parallel iteration
- ignore — gitignore-aware directory walking
- mime_guess / infer — MIME detection
- xxhash-rust — xxh3 content hashing (mmap, zero-copy)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vlm-run/mm](https://github.com/vlm-run/mm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
