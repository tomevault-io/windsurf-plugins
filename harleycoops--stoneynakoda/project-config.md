---
trigger: always_on
description: This repository implements **two complementary pipelines** for building low-resource language AI models for Stoney Nakoda:
---

# Stoney Nakoda Pipeline Operations Guide

## Repository Scope & Mission

This repository implements **two complementary pipelines** for building low-resource language AI models for Stoney Nakoda:

1. **Dictionary Fine-tuning Pipeline** - Supervised learning via OpenAI fine-tuning
   - Generates 150K Q&A pairs from bilingual dictionaries
   - Converts to OpenAI fine-tuning format (120K train / 30K validation)
   - Fine-tunes GPT models via OpenAI API

2. **Grammar RL Pipeline** - Reinforcement learning with grammar-based rewards
   - Extracts grammar rules from PDF using vision models
   - Organizes and curates rules by confidence and category
   - Generates RL training tasks for GRPO/prime-rl

**Operational Philosophy:** Run both pipelines end-to-end and **log all failures per stage** before attempting fixes. Document errors with context (command, timestamp, observed behavior) to build institutional knowledge.

---

## Environment & Secrets Checklist

### Required API Keys

Create a `.env` file (copy from `.env.example`) with the following **required** keys:

```bash
# REQUIRED: OpenAI API key for fine-tuning + grammar extraction
OPENAI_API_KEY=sk-your-openai-api-key-here

# REQUIRED: Google Gemini API key for Q&A generation
GOOGLE_API_KEY=your-google-api-key-here
```

### Optional Integration Keys

```bash
# OPTIONAL: Hugging Face dataset publishing
HUGGINGFACE_PUBLISH=false
HUGGINGFACE_TOKEN=hf_your-hf-token-here
HUGGINGFACE_DATASET_REPO=username/stoney-nakoda-dataset
HUGGINGFACE_DATASET_PRIVATE=true
ALLOW_PUBLIC_DATASET_UPLOAD=false

# OPTIONAL: Weights & Biases experiment tracking
WANDB_API_KEY=your-wandb-api-key-here
WANDB_PROJECT=stoney-nakoda-finetuning
WANDB_ENTITY=your-username-or-team
WANDB_RUN_NAME=stoney-run-001
```

### Model Override Configuration

**IMPORTANT:** These models have cost and capability implications. Update this file when changing models.

```bash
# Purpose-specific OpenAI model references
# OPENAI_MODEL is ignored by current code paths.
OPENAI_CHAT_MODEL=gpt-4.1-mini-2025-04-14
OPENAI_FINETUNE_MODEL=gpt-4.1-mini-2025-04-14
OPENAI_EXTRACTION_MODEL=gpt-5
OPENAI_TASK_MODEL=gpt-5

# Gemini model for Q&A generation (default: gemini-2.5-pro)
# Options: gemini-2.5-pro, gemini-2.0-flash-exp, gemini-1.0-pro
GEMINI_QA_MODEL=gemini-2.5-pro
```

**Cost Estimates by Model:**
- `gpt-4o-mini`: $0.150/1M input tokens, $0.600/1M output tokens
- `gpt-3.5-turbo`: $0.500/1M input tokens, $1.500/1M output tokens
- `gpt-4`: $10.00/1M input tokens, $30.00/1M output tokens
- `gpt-5` (Responses API): Varies, typically $5-10/1M tokens
- `gemini-2.5-pro`: $1.25/1M input tokens, $5.00/1M output tokens
- `gemini-2.0-flash-exp`: $0.00/1M tokens (experimental, free tier)

---

## Data-Handling Policy

### DO NOT Commit These Directories

The following directories contain generated artifacts and should **NEVER** be committed to version control:

```
Dictionaries/bilingual_training_set*.jsonl
Dictionaries/checkpoints/
OpenAIFineTune/*.jsonl
data/grammar_pages/
data/grammar_extracted_stoney/
data/rl_training_rules_stoney.json
data/training_datasets_stoney.jsonl
```

**Why:** These files are pipeline outputs that can be regenerated. They are large (150K+ lines) and change frequently during development.

### External Artifact Recording

When documenting pipeline failures:
1. **DO NOT** commit the full output files
2. **DO** record file sizes, line counts, and sample content (first 10 lines)
3. **DO** capture error logs and command outputs in issue descriptions
4. **DO** note timestamps and environment variables used

Example failure documentation:
```
Stage: Q&A Generation
Command: python bilingual_qa_generator2.py
Timestamp: 2025-10-28 17:30:00 UTC
Error: google.api_core.exceptions.ResourceExhausted: 429 Quota exceeded
Output size: 45,234 lines generated before failure
Checkpoint: Dictionaries/checkpoints_v2/checkpoint_45.jsonl exists
Mitigation: Reduced context_size from 6 to 4, resumed from checkpoint
```

---

## Pipeline Execution Steps

### Setup (One-time)

```bash
# 1. Clone repository and navigate
cd StoneyNakoda

# 2. Create virtual environment
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure API keys
cp .env.example .env
# Edit .env and add your API keys

# CHECKPOINT: Verify environment
python -c "import openai, google.generativeai; print('Dependencies OK')"
```

**Failure Checkpoint:** If imports fail, check `pip list` and compare against `requirements.txt`. Document missing packages.

### Dictionary Fine-tuning Pipeline

#### Stage 1: Q&A Generation (2-4 hours, $5-15)

```bash
python bilingual_qa_generator2.py
```

**Expected Output:**
- `Dictionaries/bilingual_training_set_v2.jsonl` (10K Q&A pairs, 5K per language)
- `Dictionaries/checkpoints_v2/checkpoint_*.jsonl` (progress checkpoints every 1000 pairs)

**Failure Checkpoints:**
- After 1000 pairs: Verify checkpoint file exists and contains valid JSON
- If interrupted: Resume automatically from last checkpoint

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HarleyCoops/StoneyNakoda](https://github.com/HarleyCoops/StoneyNakoda) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
