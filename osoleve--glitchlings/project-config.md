---
trigger: always_on
description: After completing a task, always:
---

# Glitchlings - AGENTS.md

## Quality Gates

After completing a task, always:

1. Lint with `ruff check .`
2. Type check `src/` with `python -m mypy --config-file pyproject.toml src`
3. Build the project with `uv build`
4. Run tests with `pytest`

## Repository Tour

- **`src/glitchlings/`** - Package entry point and CLI wiring.
  - `__init__.py` exposes the public API (Auggie builder, glitchlings, `Gaggle`, `summon`, `AttackConfig` helpers, `SAMPLE_TEXT`, `TranscriptTarget`).
  - `__main__.py` routes `python -m glitchlings` to the CLI entry point in `main.py`.
  - `main.py` implements the CLI: parser construction, attack config loading, glitchling summoning, and optional diff/report output.
  - `auggie.py` provides the fluent roster builder; `constants.py`/`runtime_config.py` hold defaults.
- **`src/glitchlings/attack/`** - Attack orchestrator and tokenization/metrics helpers.
  - `core.py` defines `Attack`, `AttackResult`, and tokenizer/metric resolution.
  - `core_planning.py` (pure) builds attack plans; `core_execution.py` executes plans through tokenizers and glitchlings.
  - `analysis.py` provides `SeedSweep`, `GridSearch`, and `TokenizerComparison` tools for parameter exploration.
  - `compose.py`, `encode.py`, and `metrics_dispatch.py` are pure helpers used by the reports.
  - `tokenization.py` and `metrics.py` handle impure tokenizer loading and Rust metric bridges.
  - `tokenizer_metrics.py` provides tokenizer analysis metrics (compression ratio, vocabulary coverage).
- **`src/glitchlings/zoo/`** - Core glitchling implementations and orchestration.
  - `core.py` houses `Glitchling`/`Gaggle`, dataset helpers, transcript targeting, and pipeline caching.
  - `core_planning.py` (pure) builds execution plans and normalises pipeline descriptors; `core_execution.py` dispatches plans through the Rust pipeline or Python fallbacks.
  - `corrupt_dispatch.py` (pure) resolves transcript targets and assembles corruption results; `rng.py` handles seed derivation.
  - Glitchlings: Typogre, Hokey, Mim1c, Wherewolf, Pedant (`zoo/pedant/`), Jargoyle, Rushmore (duplication/adjacent swap/zero-width), Redactyl, Scannequin, Zeedub.
- **`src/glitchlings/util/`** - Shared helpers including `SAMPLE_TEXT`, keyboard neighbour and shift maps, transcript helpers, and diff utilities.
  - `adapters.py` provides `coerce_gaggle()` for normalizing glitchling inputs across DLC integrations.
- **`src/glitchlings/protocols.py`** - Protocol definitions for dependency inversion (e.g., `Corruptor` protocol allows attack module to work with glitchlings without circular imports).
- **`src/glitchlings/assets/`** - Bundled data (homoglyphs, homophones, Hokey assets, OCR confusions, pipeline assets) plus lexeme dictionaries under `lexemes/` (synonyms, colors, corporate, academic, cyberpunk, lovecraftian).
- **`src/glitchlings/conf/`** - Configuration schema, dataclasses, and loaders for YAML attack configs.
- **`src/glitchlings/compat/`** - Optional dependency loaders (datasets, tokenizers, PyTorch, Lightning, Hugging Face).
- **`src/glitchlings/dev/`** - Doc refresh helpers (`python -m glitchlings.dev.docs` / `glitchlings-refresh-docs`).
- **`src/glitchlings/dlc/`** - Optional DLC integrations.
  - `_shared.py` provides shared utilities for dataset column resolution and batch corruption.
  - `prime.py` integrates with the `verifiers` environments and Prime/HF connectors.
  - `pytorch.py` and `pytorch_lightning.py` provide PyTorch Dataset/DataModule wrappers.
  - `huggingface.py` wraps Hugging Face datasets with corruption transforms.
  - `langchain.py` provides LangChain integration helpers.
  - `nemo.py` provides NVIDIA NeMo DataDesigner column generator plugin.
  - `gutenberg.py` provides access to the Gutenberg corpus.
- **`benchmarks/`** - Performance harnesses (`pipeline_benchmark.py`) covering Python and Rust execution paths.
- **`docs/`** - Field guide, development notes, CLI/Attack/config docs, and generated references (`cli.md`, `configuration.md`, `attack.md`, `monster-manual.md`, `glitchling-gallery.md`). Regenerate generated pages with `python -m glitchlings.dev.docs`.
- **`tests/`** - Pytest suite covering orchestration, determinism, DLC hooks, CLI, and Rust parity.
  - Highlights: `tests/core/test_core_planning.py` (plan building/pipeline descriptors), `tests/core/test_corrupt_dispatch.py` (transcript targeting), `tests/attack/test_attack.py` (Attack orchestration, tokenization, metrics), `tests/core/test_hybrid_pipeline.py` (Rust pipeline parity), `tests/cli/test_cli.py` (CLI contract), `tests/dlc/test_prime_echo_chamber.py` (Prime DLC), `tests/core/test_parameter_effects.py` (argument coverage).

## Coding Conventions

- Target **Python 3.10+** (see `pyproject.toml`).
- Follow the import order used in the package: standard library, third-party, then local modules.
- Every new glitchling must:
  - Subclass `Glitchling`, setting `scope` and `order` via `AttackWave` / `AttackOrder` from `core.py`.
  - Accept keyword-only parameters in `__init__`, forwarding them through `super().__init__` so they are tracked by `set_param`.
  - Drive all randomness through the instance's RNG and the boundary helpers in `zoo.rng`; do not rely on module-level RNG state.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [osoleve/glitchlings](https://github.com/osoleve/glitchlings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
