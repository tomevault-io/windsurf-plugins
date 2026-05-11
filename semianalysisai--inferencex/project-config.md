---
trigger: always_on
description: This file provides guidance for AI agents working with the InferenceX codebase.
---

# AGENT.md

This file provides guidance for AI agents working with the InferenceX codebase.

## Project Overview

InferenceX is an open-source, automated benchmarking system that continuously tracks LLM inference performance across different hardware platforms (NVIDIA B200/H100/H200/GB200, AMD MI300X/MI325X/MI355X) and software stacks (vLLM, SGLang, TensorRT-LLM, ATOM). Results are published to https://inferencex.com/.

## Directory Structure

```
├── benchmarks/              # Shell scripts for running benchmarks
│   ├── benchmark_lib.sh     # Shared benchmarking/eval utilities
│   ├── dsr1_*.sh            # Deepseek R1-specific benchmark scripts
│   └── gptoss_*.sh          # gptoss-specific benchmark scripts
├── runners/                 # Launch scripts for different hardware
│   ├── launch_b200/h100/h200-*.sh     # NVIDIA launcher scripts
│   └── launch_mi*.sh                  # AMD launcher scripts
├── utils/                   # Python utilities
│   ├── matrix_logic/        # Config generation and validation
│   │   ├── generate_sweep_configs.py  # CLI for generating benchmark matrix
│   │   ├── validation.py              # Pydantic validation models
│   │   └── test_*.py                  # Unit tests
│   ├── bench_serving/       # Benchmark serving client (upstreamed from vLLM)
│   │   ├── benchmark_serving.py       # Main benchmark client script
│   │   ├── backend_request_func.py    # Backend-specific request functions
│   │   └── benchmark_utils.py         # Utility functions
│   ├── evals/               # Eval task definitions for lm-eval
│   │   ├── EVALS.md         # Evals documentation
│   │   ├── gsm8k.yaml
│   │   └── gpqa_diamond.yaml
│   ├── collect_eval_results.py  # Aggregates eval results
│   ├── process_result.py    # Post-processes benchmark results
│   ├── process_changelog.py # Processes perf-changelog.yaml
│   └── summarize.py         # Generates markdown summaries
├── .github/
│   ├── workflows/           # GitHub Actions CI/CD
│   │   ├── run-sweep.yml    # Main performance sweep
│   │   ├── e2e-tests.yml    # End-to-end testing
│   │   ├── benchmark-tmpl.yml           # Single-node benchmark job template
│   │   ├── benchmark-multinode-tmpl.yml # Multi-node benchmark job template
│   │   └── collect-evals.yml            # Eval results collection
│   └── configs/             # Master configuration files
│       ├── nvidia-master.yaml
│       ├── amd-master.yaml
│       └── runners.yaml
└── perf-changelog.yaml      # Triggers benchmarks on changes
```

## Terminology

- **STP (Single Token Prediction)**: Standard autoregressive decoding where one token is generated per forward pass. No speculative decoding or MTP (Multi-Token Prediction) is used. When a benchmark is labeled "STP only", it means vanilla decoding without any speculation.
- **MTP (Multi-Token Prediction)**: A technique where the model predicts multiple tokens per forward pass, typically using speculative decoding methods like EAGLE or NEXTN.

## Key Technologies

- **Python 3.13**: Core automation and config generation
- **Pydantic**: Configuration validation (V2 with strict mode)
- **Bash**: Benchmark execution and infrastructure orchestration
- **YAML**: Configuration files
- **GitHub Actions**: CI/CD workflows
- **Evals**: lm-eval validation of benchmark results
- **pytest**: Testing framework

## Development Workflow

### Running Tests

```bash
cd utils
python -m pytest matrix_logic/ -v
```

### Generating Benchmark Configs

```bash
# Full sweep with all configs
python utils/matrix_logic/generate_sweep_configs.py full-sweep \
  --config-files .github/configs/nvidia-master.yaml

# Filter by model prefix (dsr1 or gptoss)
python utils/matrix_logic/generate_sweep_configs.py full-sweep \
  --config-files .github/configs/nvidia-master.yaml \
  --model-prefix dsr1

# Filter by framework (sglang, trt, vllm, atom, dynamo-trt, dynamo-sglang)
python utils/matrix_logic/generate_sweep_configs.py full-sweep \
  --config-files .github/configs/nvidia-master.yaml \
  --framework sglang

# Filter by precision (fp4, fp8)
python utils/matrix_logic/generate_sweep_configs.py full-sweep \
  --config-files .github/configs/nvidia-master.yaml \
  --precision fp8

# Filter by runner type (b200, h100, h200, gb200, mi300x, mi325x, mi355x)
python utils/matrix_logic/generate_sweep_configs.py full-sweep \
  --config-files .github/configs/nvidia-master.yaml \
  --runner-type b200
```

### Processing Results

```bash
python utils/process_result.py
python utils/summarize.py
```

## Supported Configuration Values

When working with benchmark configurations, use these valid values:

**Models (model-prefix)**:
- `dsr1` - DeepSeek-R1-0528
- `dsv4` - DeepSeek-V4-Pro
- `gptoss` - GPT-OSS-120B

**Precisions**:
- `fp4`
- `fp8`

**Frameworks**:
- `sglang` - SGLang inference engine
- `trt` - TensorRT-LLM
- `vllm` - vLLM inference engine
- `atom` - AMD ATOM framework
- `dynamo-trt` - NVIDIA Dynamo with TensorRT-LLM backend
- `dynamo-sglang` - NVIDIA Dynamo with SGLang backend
- `sglang-disagg` - SGLang disaggregated inference

**Runners (NVIDIA)**:
- `b200` - NVIDIA B200 GPU
- `b200-trt` - NVIDIA B200 with TensorRT
- `h100` - NVIDIA H100 GPU
- `h200` - NVIDIA H200 GPU
- `gb200` - NVIDIA GB200 (multi-node)

**Runners (AMD)**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SemiAnalysisAI/InferenceX](https://github.com/SemiAnalysisAI/InferenceX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
