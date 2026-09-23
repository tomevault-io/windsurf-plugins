---
trigger: always_on
description: **verl** is a flexible, efficient, and production-ready distributed RL training library for large language models (LLMs), initiated by ByteDance Seed team and maintained by the verl community. It is the open-source version of the **HybridFlow** paper (EuroSys 2025).
---

# verl: Volcano Engine Reinforcement Learning for LLMs

## Project Overview

**verl** is a flexible, efficient, and production-ready distributed RL training library for large language models (LLMs), initiated by ByteDance Seed team and maintained by the verl community. It is the open-source version of the **HybridFlow** paper (EuroSys 2025).

The library enables:
- Easy extension of diverse RL algorithms (PPO, GRPO, ReMax, REINFORCE++, RLOO, DAPO, etc.)
- Seamless integration with existing LLM infrastructure (FSDP, Megatron-LM, vLLM, SGLang)
- Flexible device mapping for efficient resource utilization
- Support for vision-language models (VLMs) and multi-modal RL
- Multi-turn tool calling and agentic RL
- Scalability up to 671B models and hundreds of GPUs

## Technology Stack

### Core Dependencies
- **Python**: >= 3.10
- **Deep Learning Framework**: PyTorch with FSDP/FSDP2 or Megatron-LM
- **Distributed Computing**: Ray (>= 2.41.0) for distributed task orchestration
- **Inference Engines**: vLLM (>= 0.8.5, <= 0.11.0), SGLang (0.5.2), or Hugging Face Transformers
- **Data Management**: TensorDict (>= 0.8.0, <= 0.10.0), PyArrow (>= 19.0.0)
- **Configuration**: Hydra-core for configuration management
- **Experiment Tracking**: Weights & Biases, TensorBoard, MLflow, SwanLab

### Optional Dependencies
- **GPU Acceleration**: Flash Attention 2, Liger Kernel
- **Math/Science Tasks**: math-verify, latex2sympy2_extended, mathruler
- **Vision Models**: torchvision, qwen_vl_utils
- **LoRA Training**: PEFT
- **ModelScope**: For users in China (set `VERL_USE_MODELSCOPE=true`)

## Project Structure

```
verl/                       # Main source code
├── protocol.py             # Core DataProto class for data transfer between modules
├── base_config.py          # Base configuration class with dict-like interface
├── version/                # Version information
├── models/                 # Model implementations
│   ├── transformers/       # HF Transformers integration
│   ├── llama/             # Llama-specific implementations
│   ├── qwen2/             # Qwen2-specific implementations
│   └── mcore/             # Megatron-LM integration
├── trainer/               # Training algorithms and entry points
│   ├── config/            # Hydra configuration files (YAML)
│   ├── ppo/               # PPO trainer implementation
│   ├── main_ppo.py        # PPO entry point
│   ├── main_generation.py # Generation entry point
│   └── fsdp_sft_trainer.py # SFT trainer
├── workers/               # Worker implementations for distributed training
│   ├── fsdp_workers.py    # FSDP-based workers (actor, critic, reference)
│   ├── megatron_workers.py # Megatron-based workers
│   ├── actor/             # Actor-specific implementations
│   ├── critic/            # Critic-specific implementations
│   ├── rollout/           # Rollout/generation implementations
│   └── sharding_manager/  # Model sharding management
├── single_controller/     # Ray-based distributed controller
│   ├── ray/              # Ray-specific implementations
│   └── base/             # Base controller abstractions
├── utils/                 # Utility functions
│   ├── reward_score/     # Reward computation for various tasks
│   ├── checkpoint/       # Checkpoint saving/loading
│   └── kernel/           # Custom CUDA kernels
├── experimental/         # Experimental features
│   ├── agent_loop/       # Multi-turn agent loops
│   └── reward/           # Experimental reward managers
└── third_party/          # Third-party integrations
    ├── vllm/            # vLLM-specific patches
    └── sglang/          # SGLang-specific patches

tests/                    # Test suite
├── special_distributed/  # Multi-GPU required tests
├── special_e2e/         # End-to-end training tests
├── special_sanity/      # Quick sanity checks
├── special_npu/         # NPU-specific tests
├── trainer/             # Trainer unit tests
├── workers/             # Worker unit tests
└── *_on_cpu.py          # CPU-only tests

examples/                 # Example training scripts
├── ppo_trainer/         # PPO examples
├── grpo_trainer/        # GRPO examples
├── sft/                 # SFT examples
├── sglang_multiturn/    # Multi-turn RL examples
└── data_preprocess/     # Data preprocessing scripts

recipe/                  # Research algorithm implementations
├── dapo/               # DAPO algorithm
├── prime/              # PRIME algorithm
├── sppo/               # Self-play preference optimization
└── ...                 # Other algorithms

docs/                   # Documentation (Sphinx-based)
.github/workflows/      # CI/CD configurations
```

## Build and Installation

### Basic Installation
```bash
# Clone the repository
git clone https://github.com/volcengine/verl
cd verl

# Basic installation (CPU/GPU compatible)
pip install -e .

# With test dependencies
pip install -e .[test]

# With vLLM support
pip install -e .[test,vllm]

# With SGLang support
pip install -e .[test,sglang]

# Full installation with all optional dependencies
pip install -e .[test,vllm,sglang,gpu,math,geo]
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BytedTsinghua-SIA/Direct-OPD](https://github.com/BytedTsinghua-SIA/Direct-OPD) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
