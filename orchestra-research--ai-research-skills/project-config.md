---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**AI Research Skills Library** - A comprehensive open-source library of 98 AI research skills enabling AI agents to autonomously conduct AI research — from idea to paper. Each skill provides expert-level guidance (200-500 lines) with real code examples, troubleshooting guides, and production-ready workflows.

**Mission**: Enable AI agents to autonomously conduct AI research from hypothesis to experimental verification, covering the full lifecycle: literature survey, ideation, dataset preparation, training pipelines, model deployment, evaluation, and paper writing.

## Repository Architecture

### Directory Structure (98 Skills Across 23 Categories)

Skills are organized into numbered categories representing the AI research lifecycle:

- `0-autoresearch-skill/` - **Autonomous research orchestration** (1 skill: Autoresearch — central layer that manages the full lifecycle and routes to all other skills)
- `01-model-architecture/` - Model architectures (5 skills: TorchTitan, LitGPT, Mamba, RWKV, NanoGPT)
- `02-tokenization/` - Tokenizers (2 skills: HuggingFace Tokenizers, SentencePiece)
- `03-fine-tuning/` - Fine-tuning frameworks (4 skills: Axolotl, LLaMA-Factory, Unsloth, PEFT)
- `04-mechanistic-interpretability/` - Interpretability tools (4 skills: TransformerLens, SAELens, NNsight, Pyvene)
- `05-data-processing/` - Data curation (2 skills: Ray Data, NeMo Curator)
- `06-post-training/` - RLHF/DPO/GRPO (8 skills: TRL, GRPO, OpenRLHF, SimPO, verl, slime, miles, torchforge)
- `07-safety-alignment/` - Safety and guardrails (4 skills: Constitutional AI, LlamaGuard, NeMo Guardrails, Prompt Guard)
- `08-distributed-training/` - Distributed systems (6 skills: Megatron-Core, DeepSpeed, FSDP, Accelerate, PyTorch Lightning, Ray Train)
- `09-infrastructure/` - Cloud compute (3 skills: Modal, SkyPilot, Lambda Labs)
- `10-optimization/` - Optimization techniques (7 skills: Flash Attention, bitsandbytes, GPTQ, AWQ, HQQ, GGUF, ML Training Recipes)
- `11-evaluation/` - Benchmarking (3 skills: lm-evaluation-harness, BigCode, NeMo Evaluator)
- `12-inference-serving/` - Inference engines (4 skills: vLLM, TensorRT-LLM, llama.cpp, SGLang)
- `13-mlops/` - Experiment tracking (4 skills: Weights & Biases, MLflow, TensorBoard, SwanLab)
- `14-agents/` - Agent frameworks (5 skills: LangChain, LlamaIndex, CrewAI, AutoGPT, A-Evolve)
- `15-rag/` - Retrieval-augmented generation (5 skills: Chroma, FAISS, Sentence Transformers, Pinecone, Qdrant)
- `16-prompt-engineering/` - Structured output (4 skills: DSPy, Instructor, Guidance, Outlines)
- `17-observability/` - LLM observability (2 skills: LangSmith, Phoenix)
- `18-multimodal/` - Vision and speech (10 skills: CLIP, Whisper, LLaVA, Stable Diffusion, SAM, BLIP-2, AudioCraft, Cosmos Policy, OpenPI, OpenVLA-OFT)
- `19-emerging-techniques/` - Advanced methods (6 skills: MoE Training, Model Merging, Long Context, Speculative Decoding, Knowledge Distillation, Model Pruning)
- `20-ml-paper-writing/` - Paper writing (4 skills: ML Paper Writing with LaTeX templates for NeurIPS, ICML, ICLR, ACL, AAAI, COLM; Systems Paper Writing for OSDI, NSDI, ASPLOS, SOSP; Academic Plotting; Presenting Conference Talks)
- `21-research-ideation/` - Ideation (2 skills: Research Brainstorming, Creative Thinking)
- `22-agent-native-research-artifact/` - Agent-Native Research Artifact tooling (3 skills: ARA Compiler, ARA Research Manager, ARA Rigor Reviewer — ingestion, post-task provenance recording, and Seal Level 2 epistemic review)

### Skill File Structure

Each skill follows a standardized format:
```
skill-name/
├── SKILL.md                    # Main guidance (200-600 lines with YAML frontmatter)
├── references/                 # Deep documentation (300KB+ target)
│   ├── README.md              # From official docs
│   ├── api.md                 # API reference
│   ├── tutorials.md           # Step-by-step guides
│   ├── issues.md              # Real GitHub issues & solutions
│   └── releases.md            # Version history
├── scripts/                    # Helper scripts (optional)
├── templates/                  # Code templates (optional)
└── examples/                   # Example implementations (optional)
```

## Skill Quality Standards

### YAML Frontmatter Requirements (CRITICAL)

All `SKILL.md` files MUST include YAML frontmatter with these exact fields:

```yaml
---
name: skill-name-here              # kebab-case, no quotes, gerund form preferred
description: Third-person description of what AND when to use this skill  # No quotes, max 1024 chars
version: 1.0.0                     # Semantic versioning
author: Orchestra Research         # Standard author
license: MIT                       # Standard license
tags: [Tag One, Tag Two]          # Title Case (except UPPERCASE acronyms like GRPO, TRL, RLHF)
dependencies: [pkg>=1.0.0]         # Optional, with version constraints
---
```

**Critical Rules**:
- `name`: Use gerund form (e.g., `serving-llms`, `processing-data`, `grpo-rl-training`)
- `description`: Third person ("Provides guidance for..."), include WHAT it does AND WHEN to use it

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Orchestra-Research/AI-research-SKILLs](https://github.com/Orchestra-Research/AI-research-SKILLs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
