---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains experiments and assignments from Harvard CS 2881: AI Safety, which I am auditing to deepen my understanding of AI safety risks and mitigation strategies. The course focuses on:

- **Scalable oversight**: Methods for supervising AI systems that may be more capable than their human overseers
- **Preference learning**: Techniques for learning and encoding human values and preferences
- **Alignment research**: Steering powerful models toward beneficial and harmless behaviors

Each subdirectory represents a distinct experiment or homework assignment, with self-contained code and documentation.

## Repository Structure

```
ai-alignment-research/
├── CLAUDE.md                    # This file - general guidance
├── .env.example                 # API key template (shared across experiments)
├── .env                         # Your API keys (gitignored, create from .env.example)
├── check_env.py                 # Environment verification script (shared)
├── harvard-cs-2881-hw0/         # HW0: Emergent Misalignment replication
│   ├── README_EXPERIMENT.md     # Experiment-specific documentation
│   ├── train.py                 # Training scripts
│   ├── generate.py              # Generation scripts
│   └── eval/                    # Evaluation utilities
├── harvard-cs-2881-hw1-RL/      # HW1: Prompt Prefix Optimization via RL
│   ├── README_EXPERIMENT.md     # Experiment-specific documentation
│   ├── notable_people_10k.csv   # Dataset of notable people
│   ├── src/                     # Core modules (policy, benchmarks, training)
│   └── scripts/                 # Training and analysis scripts
└── [future experiments]/        # Additional course experiments
```

## Reusable Patterns and Utilities

When creating new experiments, consider reusing these patterns from existing work:

### Model Query Interface Pattern (hw0/eval/query_utils.py)

The `ModelQueryInterface` class provides a clean abstraction for:
- Loading local or HuggingFace models with device management
- Applying chat templates consistently
- Generating responses with configurable sampling parameters
- Memory management (GPU cache clearing, 4-bit quantization support)

**Reuse this for**: Any experiment requiring model inference, comparative evaluation, or response generation.

### LLM-as-Judge Evaluation (hw0/eval/judge.py)

Structured evaluation using LLMs to score responses on multiple dimensions:
- Configurable scoring rubrics
- Batch processing of evaluation prompts
- CSV-based input/output for reproducibility
- Baseline comparison support

**Reuse this for**: Experiments requiring qualitative assessment of model outputs, alignment metrics, or comparative studies.

### LoRA Finetuning Pattern (hw0/train.py)

Complete pipeline for parameter-efficient finetuning:
- Chat format preprocessing (converts conversation JSON to model-specific templates)
- PEFT/LoRA configuration with memory-efficient options
- Modular argument parsing for hyperparameter experimentation
- Training configuration persistence

**Reuse this for**: Finetuning experiments, behavioral studies, preference learning tasks.

### Data Format Conventions

Standardize on **JSONL with chat messages** for training data:
```json
{"messages": [{"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]}
```

This format is:
- Compatible with OpenAI chat APIs
- Easily convertible to any instruction-tuning template
- Human-readable and version-controllable

### Evaluation Output Format

Use **CSV with standard fields** for evaluation results:
```
id,question,response,[additional_scoring_columns]
```

This enables:
- Easy comparison across experiments
- Integration with automated grading/CI
- Visualization and statistical analysis

## Creating New Experiments

When starting a new experiment:

1. **Create experiment directory**: `mkdir harvard-cs-2881-[assignment]/`

2. **Copy reusable utilities**: Instead of rewriting from scratch, copy and adapt:
   ```bash
   # Copy model interface if you need inference
   cp harvard-cs-2881-hw0/eval/query_utils.py new-experiment/utils/

   # Copy evaluation framework if using LLM judges
   cp harvard-cs-2881-hw0/eval/judge.py new-experiment/eval/

   # Copy training scaffold if finetuning
   cp harvard-cs-2881-hw0/train.py new-experiment/train.py
   ```

3. **Document experiment specifics**: Create `README_EXPERIMENT.md` with:
   - Experiment goal and research questions
   - Specific commands for this experiment
   - Architecture details unique to this work
   - Results and findings (as you progress)

4. **Maintain separation**: Keep experiments self-contained (dependencies, data, models) to avoid cross-contamination.

## Development Philosophy

- **Favor adaptation over reinvention**: Reuse and modify existing utilities rather than building new ones from scratch
- **Modular experiments**: Each experiment should be runnable independently
- **Reproducible by default**: Use configuration files, random seeds, and clear documentation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/burnssa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
