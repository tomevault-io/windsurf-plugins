---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**Chatter**

A Rust CLI tool that wraps Qwen3-TTS from Hugging Face to provide text-to-speech capabilities with voice profile management. Users can design custom voices from natural language descriptions, clone voices from audio samples, and generate speech from text or documents — all from the terminal with progress feedback.

**Core Value:** Users can create reusable voice profiles and generate high-quality speech from text or documents without leaving the command line.

### Constraints

- **Tech stack**: Rust CLI with PyO3 for Python interop — required because model is Python-only
- **Hardware**: Apple Silicon (MLX/MPS) or CUDA-capable GPU required for local inference
- **Distribution**: `brew install chatter` must work out of the box. Chatter manages its own Python venv at `~/.local/share/chatter/venv/` with auto-setup on first run. Users do NOT manually install Python packages.
- **Dependencies**: Python 3.13 runtime (Homebrew formula declares `depends_on "python@3.13"`). `qwen-tts` is auto-installed into the managed venv on first run. Python 3.13 aligns ChatterBox (`chatterbox-tts`) with NumPy 2.x pins (PyPI allows NumPy 2.x only for Python ≥3.13).
- **Audio**: MP3 output (requires encoding from WAV produced by model)
<!-- GSD:project-end -->

<!-- GSD:stack-start source:research/STACK.md -->
## Technology Stack

## Recommended Stack
### Core Framework
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| Rust (edition 2024) | 1.85+ | Language | Systems-level performance, strong type system, excellent CLI ecosystem. PyO3 0.28 requires Rust >= 1.83. | HIGH |
| PyO3 | 0.28.2 | Python embedding | The only mature Rust-Python interop crate. Actively maintained (released 2026-02-18). Enables calling `qwen_tts` Python package directly from Rust without subprocess overhead. Use `auto-initialize` feature. | HIGH |
| clap | 4.5+ | CLI argument parsing | De facto standard for Rust CLIs. Derive macro API eliminates boilerplate. Powers ripgrep, bat, fd. | HIGH |
### Audio Processing
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| hound | 3.5.1 | WAV reading | Standard Rust WAV library. 7.5M+ downloads. Reads WAV output from qwen-tts model inference before MP3 encoding. | HIGH |
| mp3lame-encoder | 0.2.2 | WAV-to-MP3 encoding | High-level safe Rust bindings to LAME. Statically links LAME so no runtime dependency. The most ergonomic MP3 encoding option in Rust. | MEDIUM |
### File Parsing
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| pdf-extract | 0.10.0 | PDF text extraction | Purpose-built for text extraction (not PDF manipulation). Simpler API than lopdf for our read-only use case. | MEDIUM |
| pulldown-cmark | 0.13.3 | Markdown parsing | CommonMark-compliant, streaming parser. Very fast, no AST allocation needed -- we just need to strip markup and extract plain text. Released 2026-03-22. | HIGH |
### Terminal UX
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| indicatif | 0.18.4 | Progress bars | The standard Rust progress bar library. Supports bounded bars (file processing) and spinners (model loading). MultiProgress for concurrent operations. Released 2026-02-14. | HIGH |
| console | 0.15+ | Terminal utilities | Sister crate to indicatif (same `console-rs` org). Handles terminal width detection, styling, and ANSI support. | HIGH |
| owo-colors | 4.x | Colored output | Zero-allocation, no_std-compatible terminal coloring. Respects NO_COLOR/FORCE_COLOR env vars. Recommended by Rust CLI best practices guide over `colored` crate. | HIGH |
### Configuration & Serialization
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| serde | 1.x | Serialization framework | De facto standard. Required by every config/data format crate. Use `derive` feature. | HIGH |
| serde_json | 1.x | JSON serialization | Voice profile metadata storage format. Human-readable, easy to debug. | HIGH |
| toml | 0.8+ | TOML config files | Optional: if app-level config is needed beyond voice profiles. Idiomatic in Rust ecosystem. | HIGH |
| directories | 6.0.0 | XDG directory paths | Cross-platform (Linux/macOS/Windows) standard directory resolution. Returns `~/.config/chatter/` on Linux, `~/Library/Application Support/` on macOS. Actively maintained under `xdg-rs` org. | HIGH |
### Error Handling
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|
| anyhow | 1.x | Application error handling | For main.rs and top-level CLI code. Ergonomic error context with `.context()`. Standard for Rust CLI apps. | HIGH |
| thiserror | 2.x | Typed error definitions | For internal library modules (PyO3 bridge, audio encoding). Gives callers structured error types to match on. | HIGH |
### Python Environment
| Technology | Version | Purpose | Why | Confidence |
|------------|---------|---------|-----|------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [isa/chatter](https://github.com/isa/chatter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
