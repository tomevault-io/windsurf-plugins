---
trigger: always_on
description: `qr-sampler` is an engine-agnostic framework that replaces standard LLM token sampling with external-entropy-driven selection. It fetches random bytes from any entropy source (QRNGs via gRPC, OS randomness, CPU timing jitter, OpenEntropy), amplifies the signal into a uniform float via z-score or ECDF statistics, and uses that float to select a token from a probability-ordered CDF.
---

# CLAUDE.md -- Codebase Guide for Coding Agents

## What this project is

`qr-sampler` is an engine-agnostic framework that replaces standard LLM token sampling with external-entropy-driven selection. It fetches random bytes from any entropy source (QRNGs via gRPC, OS randomness, CPU timing jitter, OpenEntropy), amplifies the signal into a uniform float via z-score or ECDF statistics, and uses that float to select a token from a probability-ordered CDF.

The core sampling pipeline (`qr_sampler.core`) has **zero inference-engine dependencies** -- it operates on numpy arrays and knows nothing about torch, vLLM, or any specific engine. Engine-specific integration is handled by thin adapter classes (`qr_sampler.engines`). A vLLM V1 adapter ships out of the box; other engines (e.g., vLLM-Metal for Apple Silicon) are supported via the `EngineAdapter` plugin system and declarative YAML profiles.

The primary use case is consciousness-research: studying whether conscious intent can influence quantum-random processes in LLM token selection.

## Commands

```bash
# Run all tests
pytest tests/ -v

# Run specific test modules
pytest tests/test_config.py -v
pytest tests/test_amplification/ -v
pytest tests/test_temperature/ -v
pytest tests/test_selection/ -v
pytest tests/test_logging/ -v
pytest tests/test_entropy/ -v
pytest tests/test_processor.py -v
pytest tests/test_statistical_properties.py -v
pytest tests/test_core/ -v
pytest tests/test_engines/ -v
pytest tests/test_profiles/ -v
pytest tests/test_cli/ -v

# Run with coverage
pytest tests/ -v --cov=src/qr_sampler --cov-report=term-missing

# Install in editable mode
pip install -e .

# Install with dev dependencies
pip install -e ".[dev]"

# Install with CLI (click + jinja2)
pip install -e ".[cli]"

# Lint and format
ruff check src/ tests/
ruff format --check src/ tests/

# Type check
mypy --strict src/

# CLI commands (requires [cli] extra)
qr-sampler list engines              # List available engine profiles
qr-sampler list models --engine vllm # List known-working models for an engine
qr-sampler list entropy-sources      # List entropy source profiles
qr-sampler list amplifiers           # List amplifier profiles
qr-sampler list samplers             # List adaptive sampler profiles
qr-sampler info engine vllm          # Detailed info for a component
qr-sampler validate --engine vllm --model Qwen/Qwen2.5-1.5B-Instruct  # Check compatibility
qr-sampler build --engine vllm --entropy quantum_grpc --output ./deploy # Generate Docker Compose
```

## File map

```
src/qr_sampler/
+-- __init__.py                    # Package version (setuptools-scm), re-exports
+-- __main__.py                    # CLI entry: `python -m qr_sampler` -> cli/main.py
+-- config.py                      # QRSamplerConfig (pydantic BaseSettings), resolve_config(), validate_extra_args()
+-- exceptions.py                  # QRSamplerError -> {EntropyUnavailableError, ConfigValidationError, SignalAmplificationError, TokenSelectionError}
+-- processor.py                   # Re-export: VLLMAdapter as QRSamplerLogitsProcessor (backward compat)
+-- py.typed                       # PEP 561 marker
+-- core/                          # Engine-agnostic sampling pipeline (NO torch/vLLM imports)
|   +-- __init__.py                # Re-exports SamplingPipeline, SamplingResult, build_pipeline, etc.
|   +-- pipeline.py                # SamplingPipeline class + factory functions (build_pipeline, build_entropy_source, config_hash, accepts_config)
|   +-- types.py                   # SamplingResult frozen dataclass (token_id, one_hot, record)
+-- engines/                       # Engine adapter layer
|   +-- __init__.py                # Re-exports EngineAdapter, EngineAdapterRegistry
|   +-- base.py                    # EngineAdapter ABC (get_pipeline, close)
|   +-- registry.py                # EngineAdapterRegistry (decorator + qr_sampler.engine_adapters entry-point discovery)
|   +-- vllm.py                    # VLLMAdapter: vLLM V1 LogitsProcessor, delegates sampling to SamplingPipeline
+-- profiles/                      # Declarative YAML profile system (read-only metadata for CLI)
|   +-- __init__.py                # Re-exports ProfileLoader, CompatibilityChecker, CompatibilityReport
|   +-- schema.py                  # Pydantic models: EngineProfile, EntropySourceProfile, AmplifierProfile, SamplerProfile, etc.
|   +-- loader.py                  # ProfileLoader: discovers built-in + user override profiles, lazy loading with cache
|   +-- compatibility.py           # CompatibilityChecker: tri-state logic (known_working/untested/known_incompatible)
|   +-- engines/
|   |   +-- vllm.yaml              # vLLM NVIDIA GPU profile
|   |   +-- vllm_metal.yaml        # vLLM Apple Silicon / Metal profile
|   +-- entropy/
|   |   +-- system.yaml            # os.urandom() entropy
|   |   +-- quantum_grpc.yaml      # gRPC quantum entropy
|   |   +-- timing_noise.yaml      # CPU timing jitter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Entropic-Science/qr-sampler](https://github.com/Entropic-Science/qr-sampler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
