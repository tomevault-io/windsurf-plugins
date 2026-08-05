---
trigger: always_on
description: Companion to `LEARNINGS.md` (non-obvious lessons + a labelled history
---

# AGENTS.md — Codebase Guide for Coding Agents

Companion to `LEARNINGS.md` (non-obvious lessons + a labelled history
section) and `README.md` (end-user documentation).

## What this project is

`qr-sampler` is an engine-agnostic framework that replaces standard LLM token
sampling with external-entropy-driven selection. It fetches random bytes from
any entropy source (QRNGs via gRPC, OS randomness, CPU timing jitter,
OpenEntropy), amplifies the signal into a uniform float via z-score or ECDF
statistics, and uses that float to select a token from a probability-ordered
CDF. The primary use case is weak-signal integration research: studying
whether small statistical biases in physical entropy sources are detectable
in LLM token selection. In server-draw mode the integration happens
server-side (Qbert0G's `qr_purity.PurityService`) and each draw arrives with
purity/coherence metadata; the research narrative lives in Qbert0G's README.

Two independent consumers sit on top of the library:

1. **vLLM** loads `VLLMAdapter` as a V1 logits processor via the
   `vllm.logits_processors` entry point (no vLLM source changes).
2. **`qr-llm-qthought`** (sibling checkout at
   `../Entropic-Science/qr-llm-qthought`, relative to this repo) imports the
   `QthoughtRoller` entropy stack — no vLLM, no GPU — through
   `qr_sampler.contract` (see below).

## Verification — one command

```bash
python scripts/check.py            # every oracle: lint, format, types, security, tests
python scripts/check.py --only lint,types
```

The oracle rows (CI and pre-commit invoke this same script):

| Check | Command |
|---|---|
| lint | `ruff check .` |
| format | `ruff format --check .` |
| types | `mypy --strict src/` |
| security | `bandit -c pyproject.toml -r src/ -q` |
| tests | `pytest tests/ -v --cov=src/qr_sampler` (coverage `fail_under=90`) |

A change is NOT done until every row passes. Use these as ground truth — do
not act as a linter yourself.

## Layering (imports only point down)

```
L0 foundation   exceptions.py, config/  (model + presets + resolve)
L1 core         core/, entropy/, amplification/, temperature/, selection/,
                logging/, telemetry/, proto/
L2 roller       qthought.py
L3 adapters     engines/
L4 periphery    cli/, profiles/, templates/, __main__.py, contract.py
```

Rules:

- Nothing in L0–L3 may import from `cli/`, `profiles/`, or `templates/` —
  the periphery is CLI/documentation tooling, never runtime sampling.
- `core/` has **zero engine dependencies**: numpy only, no torch/vLLM.
  Engine-specific code lives exclusively under `engines/`.
- `contract.py` is a pure re-export module (may import from anywhere; nothing
  internal imports it).
- Import side effects are forbidden. `import qr_sampler` and
  `import qr_sampler.engines.vllm` are 100% side-effect-free — no sockets, no
  monkey-patches, no file writes. Pinned by
  `tests/test_engines/test_import_time_socket_guard.py`.

## The cross-repo contract (`contract.py`)

`src/qr_sampler/contract.py` is **the only surface downstream consumers may
import**; `qr-llm-qthought` is the live consumer. It re-exports (grouped):

- roller + provenance: `QthoughtRoller`, `ChoiceProvenance`, `BindSpec`, `IntRange`
- config + presets: `QRSamplerConfig`, `resolve_config`, `resolve_preset`,
  `BUILTIN_PRESETS`, `PRESET_QTHOUGHT`, `PRESET_QTHOUGHT_THINK`, `PRESET_QTHOUGHT_VOICE`
- entropy primitives: `EntropySource`, `MockUniformSource`, `FallbackEntropySource`
- exceptions: `EntropyUnavailableError`, `ConfigValidationError`
- `CONTRACT_VERSION` — bump on ANY breaking change to this surface; qthought
  asserts it at import and fails loudly on mismatch.

Internal module boundaries are free to move as long as `contract.__all__`
keeps re-exporting the same names. `tests/test_contract.py` pins `__all__`,
the three qthought preset dicts (scientific lineage — do not touch their
values), and `inspect.signature` snapshots of the roller surface. Its
counterpart `tests/test_sampler_contract.py` lives in the qthought repo.
Breaking-seam changes must land atomically with the qthought consumer update
(both repos green in the same increment).

## File map

```
src/qr_sampler/
+-- __init__.py                # Version + top-level re-exports (side-effect-free)
+-- __main__.py                # `python -m qr_sampler` -> cli/main.py
+-- exceptions.py              # QRSamplerError -> {EntropyUnavailable, ConfigValidation, SignalAmplification, TokenSelection}Error
+-- contract.py                # Cross-repo seam (see above)
+-- qthought.py                # QthoughtRoller: typed random-choice family over the entropy stack (choose/coin/bind_int/draw_u/draw_index + ChoiceProvenance)
+-- py.typed
+-- config/
|   +-- model.py               # QRSamplerConfig (pydantic BaseSettings); PER_REQUEST_FIELDS derived from Field(json_schema_extra={"per_request": True})
|   +-- presets.py             # BUILTIN_PRESETS, PRESET_* name constants, resolve_preset(), expand_extra_args()
|   +-- resolve.py             # resolve_config(), validate_extra_args() — the single validation point
+-- core/
|   +-- pipeline.py            # SamplingPipeline + factories (build_pipeline, build_entropy_source, config_hash, derive_commit_nonce)
|   +-- types.py               # SamplingResult (frozen)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Entropic-Science/qr-sampler](https://github.com/Entropic-Science/qr-sampler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
