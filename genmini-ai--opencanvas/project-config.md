---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CRITICAL GLOBAL RULES

### NEVER Create Synthetic/Fallback Data
- **NEVER** create synthetic gaps, scores, or results as fallbacks
- **NEVER** use fake data to mask real issues or keep systems running
- If evaluation fails → Stop evolution, don't analyze error data
- If no gaps found → This is SUCCESS (system optimized), not failure
- Real failures should propagate and be fixed, not hidden with synthetic data
- This applies to ALL components: evaluation, reflection, improvement, implementation

## Development Commands

```bash
# Install dependencies
pip install -e .                    # Recommended: CLI + core functionality
pip install -r requirements-all.txt # Complete installation (CLI + API)
pip install -r requirements.txt     # Core functionality only
pip install -r requirements-api.txt # API dependencies only

# Install browser drivers for PDF conversion
playwright install chromium

# Setup environment variables
cp .env.example .env
# Edit .env with your API keys

# CLI Usage
opencanvas generate "AI in healthcare" --purpose "academic presentation" --theme "clean minimalist"
opencanvas convert output/slides.html --output presentation.pdf --zoom 1.5
opencanvas evaluate evaluation_folder/
opencanvas pipeline "quantum computing" --purpose "conference talk" --evaluate

# Start API server
opencanvas api --host 0.0.0.0 --port 8000 --reload

# Run tests
python run_tests.py                 # Full E2E test suite
python run_tests.py light           # Light mode (faster)
python run_tests.py topic           # Topic tests only
python run_tests.py pdf             # PDF tests only
python run_tests.py force           # Force regenerate all files

# Adversarial evaluation testing
python run_adversarial_eval_test.py
python run_adversarial_eval_test.py --regenerate

# Topic Evolution System - Autonomous presentation improvement
python topic_evolution.py --run  # Run complete autonomous evolution system (default: 2 iterations)
python topic_evolution.py --run --max-iterations 1  # Single iteration test to verify tool/prompt changes
python topic_evolution.py --run --max-iterations 1 --topic "AI in healthcare"  # Custom topic single iteration
python topic_evolution.py --run --diagnostic  # Run with diagnostic output for debugging
python topic_evolution.py --run --prompt-only  # Focus only on prompt optimization, skip tool creation
python topic_evolution.py --run --initial-prompt evolution_runs/evolved_prompts/generation_prompt_v3.txt  # Start with custom prompt
python topic_evolution.py --run --resume evolution_runs/tracked_evolution_20250815_162354  # Resume from existing experiment

# PDF Evolution System - PDF-based presentation evolution
python pdf_evolution.py --max-iterations 2 --prompt-only --test-pdfs https://arxiv.org/pdf/2505.20286
python pdf_evolution.py --max-iterations 4 --prompt-only --test-pdfs URL1 URL2 URL3 --memory

# Validate configuration
python -c "from opencanvas.config import Config; Config.validate(); print('✅ Configuration valid')"
```

## Project Architecture

This is a Python-based presentation generation and evaluation system with both CLI and REST API interfaces:

**Core Components:**
- **CLI Interface** (`src/opencanvas/main.py`): Primary command-line interface with `generate`, `convert`, `evaluate`, `pipeline`, and `api` commands
- **Generation Router** (`src/opencanvas/generators/router.py`): Routes between topic-based and PDF-based generation
- **Topic Generator** (`src/opencanvas/generators/topic_generator.py`): Creates presentations from text topics with optional web research
- **PDF Generator** (`src/opencanvas/generators/pdf_generator.py`): Extracts and converts PDF content to presentations
- **HTML-to-PDF Converter** (`src/opencanvas/conversion/html_to_pdf.py`): Converts generated HTML slides to PDF using Selenium/Playwright
- **AI Evaluator** (`src/opencanvas/evaluation/evaluator.py`): Evaluates presentation quality using Claude, GPT, or Gemini models

**Evolution System** (Autonomous Improvement):
- **Evolution System** (`src/opencanvas/evolution/core/evolution.py`): Main orchestrator for autonomous presentation quality improvement
- **Multi-Agent System** (`src/opencanvas/evolution/core/agents.py`): Reflection, improvement, and implementation agents
- **Auto Tool Implementation** (`src/opencanvas/evolution/core/tool_implementation.py`): Fully autonomous tool creation and deployment system
- **Evolved Router** (`src/opencanvas/evolution/core/evolved_router.py`): Enhanced generation router with evolved prompts and auto-generated tools
- **Prompt Evolution** (`src/opencanvas/evolution/core/prompts.py`): Dynamic prompt improvement based on evaluation results
- **Tools Manager** (`src/opencanvas/evolution/core/tools.py`): Tool discovery, specification, and lifecycle management

**API Layer:**
- **FastAPI Application** (`src/api/app.py`): REST API with auto-generated documentation
- **API Routes** (`src/api/routes.py`): Endpoints for generation, conversion, evaluation, and pipeline operations
- **Pydantic Models** (`src/api/models.py`): Request/response schemas

### Key Features


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [genmini-ai/OpenCanvas](https://github.com/genmini-ai/OpenCanvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
