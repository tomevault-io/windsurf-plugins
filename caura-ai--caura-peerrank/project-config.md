---
trigger: always_on
description: Handles both data formats:
---

# PeerRank

**LLM Peer Evaluation System** - Models generate questions, answer them with web search, cross-evaluate each other's responses, and produce a ranked report with bias analysis.

## Features

- **5-Phase Pipeline**: Question generation → Answering → Cross-evaluation → Report → Analysis
- **12 Models**: OpenAI, Anthropic, Google, xAI, DeepSeek, Together AI, Perplexity, Moonshot AI, Mistral
- **Bias Detection**: Measures self-bias, name bias, and position bias through controlled evaluation modes
- **Elo Ratings**: Alternative ranking via pairwise comparisons (K=32, excludes self-evaluations)
- **Standardized Web Grounding**: Single search per question via Tavily or SerpAPI, identical context for all models (fair comparison)
- **Cost Tracking**: Real-time token usage and cost analysis per model
- **Publication Figures**: Generate publication-quality charts and statistical analysis
- **TruthfulQA Validation**: Correlate peer rankings with ground truth accuracy
- **GSM8K Validation**: Correlate peer rankings with math accuracy (r=0.986)

## Prerequisites

- Python 3.10+
- API keys for LLM providers you want to test (see [API Keys](#api-keys))

## Installation

```bash
git clone https://github.com/caura-ai/caura-PeerRank.git
cd caura-PeerRank
pip install -e .       # Install as package
cp .env.example .env   # Add your API keys
```

Or install directly from GitHub:
```bash
pip install git+https://github.com/caura-ai/caura-PeerRank.git
```

## Quick Start

```bash
python peerrank.py              # Interactive menu
python peerrank.py --all        # Run all 5 phases
python peerrank.py --health     # Test API connectivity
```

## Commands

```bash
python peerrank.py              # Interactive menu
python peerrank.py --phase 1    # Run specific phase (1-5)
python peerrank.py --all        # Run all phases (1-5)
python peerrank.py --resume     # Resume from last completed
python peerrank.py --models gpt-5.5,claude-opus-4-7    # Include only these models
python peerrank.py --exclude gemini-3-pro-preview      # Exclude these models
python peerrank.py --categories factual,reasoning      # Include only these categories
python peerrank.py --exclude-categories creative       # Exclude these categories
python peerrank.py --seed 42    # Reproducible shuffle ordering for Phase 3
python peerrank.py --web-search on   # Enable Phase 2 web grounding (default)
python peerrank.py --web-search off  # Disable Phase 2 web grounding (test knowledge only)
python peerrank.py --web-search-3 on # Enable Phase 3 web grounding (reuse Phase 2 data)
python peerrank.py --web-search-3 off # Disable Phase 3 web grounding (default)
python peerrank.py --grounding-provider tavily  # Use Tavily for grounding (default)
python peerrank.py --grounding-provider serpapi # Use SerpAPI for grounding
python peerrank.py --judge gpt-5.2   # Select judge model for Phase 5
python peerrank.py --rev v2     # Set revision tag for output files
python peerrank.py --health     # API health check
streamlit run peerrank_ui.py    # Launch Streamlit UI
python generate_figures_PeerRank.py --revision v1 --output figures/  # Generate publication figures
python generate_figures_TFQ.py --output figures/              # Generate TFQ validation figures
python validate_gsm8k.py --all --num-questions 50                      # Run GSM8K math validation
python validate_gsm8k.py --difficulty hard --num-questions 20          # GSM8K with hard questions only
python validate_mmlu.py --all --num-questions 50                       # Run MMLU validation (all subjects)
python validate_mmlu.py --subset medical --num-questions 30            # MMLU medical domain only
python validate_mmlu.py --subset law --num-questions 30                # MMLU law domain only
```

## Interactive Menu

```
  Revision: v1  |  Progress: 0/5
  Models: 3/12  |  Categories: 5/5  |  Questions: 2/model
  P2: web=ON  |  P3: seed=rand, web-grounding=OFF  |  P5: gpt-5.2
  Grounding: TAVILY

  --- Run ---
  [1-5] Run phase    [A] All    [R] Resume    [H] Health check

  --- Setup ---
  [V] Revision    [M] Models    [N] Questions    [C] Categories    [S] Search provider

  --- Phase Settings ---
  [W] P2 web grounding  [D] P3 seed    [G] P3 web grounding
  [J] P5 judge

  [Q] Quit
```

## Architecture

```
peerrank/                      # Core package (pip installable)
  __init__.py                  # Package exports (config, models, providers, validation_utils)
  models.py                    # Model definitions and pricing (ALL_MODELS)
  config.py                    # Settings, utilities, derived model lists
  providers.py                 # LLM API calls with grounding injection
  validation_utils.py          # Shared utilities for validation scripts
peerrank.py                    # CLI entry point
peerrank_ui.py                 # Streamlit UI (live comparison)
peerrank_phase1.py             # Question generation
peerrank_phase2.py             # Answer questions (web search configurable)
peerrank_phase3.py             # Cross-evaluation (web search configurable, 3 bias modes)
peerrank_phase4.py             # Report generation
peerrank_phase5.py             # Final analysis by judge LLM
generate_figures_PeerRank.py   # Publication-quality figure generation (Figs 4-6, 10-17)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caura-ai/caura-PeerRank](https://github.com/caura-ai/caura-PeerRank) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
