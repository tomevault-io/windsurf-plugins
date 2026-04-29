---
trigger: always_on
description: **"LLM의 SQLite"** — 가장 작고, 가장 읽기 쉽고, 가장 쉽게 임베딩할 수 있는 LLM 엔진.
---

# quant.cpp — Agent Development Guide

## Project Vision

**"LLM의 SQLite"** — 가장 작고, 가장 읽기 쉽고, 가장 쉽게 임베딩할 수 있는 LLM 엔진.

Two directions:
1. **Embedding Engine**: quant.h 단일 헤더(15K LOC)로 어디든 LLM 추가
2. **KV Compression Research**: 함수 3개로 새 양자화 타입 추가 가능한 연구 플랫폼

**Non-goals**: GPU 속도 경쟁 (llama.cpp 영역), 배치 서빙 (vLLM 영역), 학습

## Project Overview

quant.cpp is a minimal C inference engine for local LLM with KV cache compression.
72K LOC, pure C, zero dependencies. Supports 7 architectures via GGUF.
Killer feature: KV cache compression — 7x compression with PPL +0.0% vs FP32.
Ships as quant.h (15K LOC single header) and WASM (192KB).

## Architecture

```
include/turboquant/   — Public C API (turboquant.h, tq_types.h, tq_spec.h)
src/core/             — Algorithms (tq_polar.c, tq_qjl.c, tq_turbo.c, tq_uniform.c, tq_traits.c)
src/cache/            — Paged cache + progressive compression
src/backend/cpu/      — CPU kernels (generic, AVX2, NEON)
src/backend/cuda/     — CUDA kernels
src/backend/metal/    — Metal compute shaders (7 kernels: matmul, rmsnorm, rope, attention, etc.)
src/engine/           — GGUF loader, transformer forward, tokenizer, generate
tests/                — Google Test unit tests (34 tests)
wasm/                 — Browser demo (quant.wasm 192KB + index.html)
docs/                 — API reference, custom quantization guide, tech report
examples/             — Embedding examples (minimal, chat, kv_compare)
bench/                — Performance benchmarks
spec/                 — Format specification + test vectors
integrations/         — llama.cpp, vLLM, Python bindings
harness/              — Autonomous development harness (run.sh, team.toml)
```

## Key Documents

- `docs/prd_v0.1.md` — Full product requirements
- `docs/wbs_v0.1.md` — Work breakdown structure with checklists
- `program.md` — Current agent task specification (READ THIS FIRST)
- `score.sh` — Automated 5-dimension scoring (0.0 ~ 1.0)

## Reference Implementations (DO NOT MODIFY)

- `refs/QJL/` — QJL Python/CUDA implementation (Amir Zandieh)
- `refs/PolarQuant/` — PolarQuant Python/Triton implementation
- `refs/llama.cpp/` — llama.cpp fork with TQ1/TQ2 weight quantization
- `refs/vllm/` — vLLM KV cache infrastructure
- `refs/onnx/` — ONNX quantization operator specification

## Build & Test

```bash
cmake -B build -DCMAKE_BUILD_TYPE=Release -DTQ_BUILD_TESTS=ON -DTQ_BUILD_BENCH=ON
cmake --build build -j$(sysctl -n hw.ncpu 2>/dev/null || nproc)
ctest --test-dir build --output-on-failure
```

## Scoring (5-Dimension Harness)

```bash
bash score.sh              # Full 5-dimension evaluation
bash score.sh --quick      # Build + correctness only (fast iteration)
bash score.sh --bench      # Performance only
bash score.sh --quality    # Quantization quality only
```

The score measures 5 dimensions:
1. **Structure** (10w) — Headers, sources, tests, specs exist
2. **Correctness** (11w) — Builds, tests pass, zero warnings
3. **Quality** (11w) — Roundtrip MSE < 0.01, attention cosine > 0.99
4. **Performance** (14w) — Throughput, compression ratio 5x, SIMD speedup 4x
5. **Integration** (6w) — llama.cpp/vLLM/Python plugins, docs, examples

## Coding Standards

- C11 for core library, C++17 for tests/CUDA/Metal wrappers
- No external dependencies in core (libc/libm only)
- Every block struct must have static_assert size verification
- Every public function must have a unit test
- ONNX LSB-first bit-packing convention for all quantized formats
- Use refs/ code as algorithm reference — port to C, don't wrap Python

## Quantization Types

| Type | Key Bits | Algorithm | Block Size |
|------|----------|-----------|------------|
| TQ_POLAR_3B | 3 | PolarQuant | 128 |
| TQ_POLAR_4B | 4 | PolarQuant | 128 |
| TQ_QJL_1B | 1 | QJL sign hash | 256 |
| TQ_TURBO_3B | 3 | Polar 2b + QJL 1b | 128 |
| TQ_TURBO_4B | 4 | Polar 3b + QJL 1b | 128 |
| TQ_UNIFORM_4B | 4 | Min-Max | 128 |
| TQ_UNIFORM_2B | 2 | Min-Max | 128 |

## Module Ownership (Conflict-Free Zones)

Each module has exclusive file ownership. When working in parallel, agents must only modify their assigned files:

| Module | Owned Files | Dependencies |
|--------|-------------|-------------|
| `foundation` | `CMakeLists.txt`, `include/**`, `src/core/tq_traits.c` | None |
| `polar` | `src/core/tq_polar.*`, `tests/test_polar.*` | foundation |
| `qjl` | `src/core/tq_qjl.*`, `tests/test_qjl.*` | foundation |
| `turbo` | `src/core/tq_turbo.*`, `tests/test_turbo.*` | polar, qjl |
| `uniform` | `src/core/tq_uniform.*`, `src/core/tq_value_quant.*`, `tests/test_uniform.*`, `tests/test_value.*` | foundation |
| `cache` | `src/cache/**`, `tests/test_paged_cache.*`, `tests/test_progressive.*` | foundation |
| `simd-neon` | `src/backend/cpu/**` | polar, qjl |
| `gpu-cuda` | `src/backend/cuda/**` | polar, qjl |
| `gpu-metal` | `src/backend/metal/**` | polar, qjl |
| `bench` | `bench/**`, `spec/**`, `tests/reference/**` | all core |
| `integration` | `integrations/**`, `bindings/**`, `examples/**` | all |

## Agent Team & Skills (Harness Architecture)

이 프로젝트는 [Harness](https://github.com/revfactory/harness) 패턴을 적용한 에이전트 팀 구조를 사용한다.

### Agents (.claude/agents/)

| Agent | Role | Subagent Type |
|-------|------|---------------|
| `architect` | 기술 리더, 작업 분해/위임, Merge Gate | Leader |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quantumaikr/quant.cpp](https://github.com/quantumaikr/quant.cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
