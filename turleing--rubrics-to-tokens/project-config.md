---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ROLL (Reinforcement Learning Optimization for Large-Scale Learning) is an efficient RL library for Large Language Models (LLMs) using large-scale GPU resources. It enhances LLM performance in human preference alignment, complex reasoning, and multi-turn agentic interactions.

## Core Development Rules

1. Code Quality
   - Type hints required for all code
   - Public APIs must have docstrings
   - Document with docstrings
   - Functions must be focused and small
   - Follow existing patterns exactly
   - Line length: 119 characters
   - Follow existing module structure when adding new workers/pipelines
   - Configuration-driven design - avoid hardcoding parameters
   - Python 3.10+ target
   - PEP 8 naming (snake_case for functions/variables)
   - Class names in PascalCase
   - Constants in UPPER_SNAKE_CASE
   - Use f-strings for formatting

## Development Philosophy

- **Simplicity**: Write simple, straightforward code
- **Readability**: Make code easy to understand
- **Performance**: Consider performance without sacrificing readability
- **Maintainability**: Write code that's easy to update
- **Testability**: Ensure code is testable
- **Reusability**: Create reusable components and functions
- **Less Code = Less Debt**: Minimize code footprint

## Coding Best Practices

- **Early Returns**: Use to avoid nested conditions
- **Descriptive Names**: Use clear variable/function names (prefix handlers with "handle")
- **Constants Over Functions**: Use constants where possible
- **DRY Code**: Don't repeat yourself
- **Functional Style**: Prefer functional, immutable approaches when not verbose
- **Minimal Changes**: Only modify code related to the task at hand
- **Function Ordering**: Define composing functions before their components
- **TODO Comments**: Mark issues in existing code with "TODO:" prefix
- **Simplicity**: Prioritize simplicity and readability over clever solutions
- **Build Iteratively** Start with minimal functionality and verify it works before adding complexity
- **Run Tests**: Test your code frequently with realistic inputs and validate outputs
- **Build Test Environments**: Create testing environments for components that are difficult to validate directly
- **Functional Code**: Use functional and stateless approaches where they improve clarity
- **Clean logic**: Keep core logic clean and push implementation details to the edges
- **File Organsiation**: Balance file organization with simplicity - use an appropriate number of files for the project scale

## Common Development Commands

### Running Pipelines
```bash
# RLVR Pipeline
python examples/start_rlvr_pipeline.py --config_name sppo_config

# Agentic Pipeline
python examples/start_agentic_pipeline.py --config_name sokoban_ppo_config

# Override config parameters
python examples/start_rlvr_pipeline.py rollout_batch_size=128 max_steps=1000
```

## High-Level Architecture

### 1. **Pipeline Definitions** (`roll/pipeline/`)
The framework provides two main training pipelines:

- **RLVR Pipeline** (Reinforcement Learning with Verifiable Rewards)
  - Multi-domain training with dynamic reward routing
  - Supports PPO, GRPO, Reinforce++ algorithms
  - Handles math, code, general reasoning, and other domains
  
- **Agentic Pipeline**
  - Environment-based RL training
  - Supports environments like Sokoban, WebShop, FrozenLake
  - Trajectory collection and policy optimization

### 2. **Distributed System** (`roll/distributed/`)
Multi-role distributed architecture using Ray:

- **Executor**: Worker management, clusters, model update groups
- **Scheduler**: Resource management, generation/reward scheduling
- **Strategy**: Multiple backend support:
  - **Megatron-Core**: Model parallelism (TP, PP, CP, EP)
  - **DeepSpeed**: ZeRO optimization, CPU offloading
  - **vLLM/SGLang**: High-throughput inference
  - **FSDP**: PyTorch native sharding
  - **HuggingFace**: Standard transformers integration

### 3. **Worker Types** (Role-based architecture)
- **Actor Workers**: Policy model training and inference
- **Critic Workers**: Value function estimation
- **Reference Workers**: KL divergence calculation
- **Reward Workers**: Domain-specific reward computation
  - Math reward models
  - Code evaluation (sandbox)
  - LLM-as-judge
  - Rule-based rewards (IFEval, CrossThinkQA)
- **Environment Workers**: For agentic tasks

### 4. **Model Support** (`roll/models/`)
- Model providers for different frameworks
- Function providers for specialized operations
- TRL (Transformers Reinforcement Learning) patches
- Vision-Language model support (Qwen-VL)

## 📁 Key Components

### Configuration System
- Uses Hydra for hierarchical YAML configs
- Supports CLI overrides
- Modular configuration for different components

### Utilities (`roll/utils/`)
- **Collective operations**: Distributed training primitives
- **Checkpoint management**: Save/resume functionality
- **Metrics tracking**: Performance monitoring
- **Code evaluation**: Sandboxed code execution

### Third-party Integrations (`roll/third_party/`)
- Custom patches for vLLM, SGLang, DeepSpeed, Megatron

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TURLEing/Rubrics-To-Tokens](https://github.com/TURLEing/Rubrics-To-Tokens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
