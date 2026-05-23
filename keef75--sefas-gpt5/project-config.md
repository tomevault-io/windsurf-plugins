---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

SEFAS (Self-Evolving Federated Agent System) is a next-generation multi-agent AI system implementing autonomous evolution through federated intelligence. The system features 17 specialized agent roles that collaborate using advanced redundancy patterns, belief propagation, and industrial-grade reliability mechanisms including N-version programming, quorum-based validation, and circuit breakers.

## Core Architecture

### Enhanced Federated Agent System
- **Agent Roles**: 17 specialized agents organized in 3 layers:
  - **Layer 1 - Strategic Command**: Orchestrator, Task Decomposer, Strategy Evolver
  - **Layer 2 - Creative & Analytical Generation**: Proposer Alpha/Beta/Gamma, Domain Expert, Technical Architect, Innovation Catalyst, Strategic Planner
  - **Layer 3 - Quality Assurance & Validation**: Logic/Semantic/Consistency Validators, Quality Assurer, Performance Optimizer, Risk Assessor, Compliance Officer
- **Enhanced Reliability**: Industrial-grade redundancy with N-version programming, quorum validation, and circuit breakers
- **Agent Factory**: Dynamic agent creation from `config/agents.yaml` with full LLM parameter control
- **Management Interface**: CLI tools for easy agent configuration and monitoring via `scripts/manage_agents.py`

### Enhanced Data Flows & Reliability
- **Redundant Proposal Generation**: N-version programming with 5 diverse agent providers for exponential error reduction
- **Belief Propagation Engine**: Advanced consensus calculation with convergence detection and confidence weighting
- **Quorum-Based Validation**: 3+ validators required for consensus with intelligent fallback mechanisms
- **Circuit Breakers**: Fault tolerance preventing cascade failures with automatic recovery
- **Hedged Requests**: Tail latency reduction through concurrent execution with first-wins semantics
- **GPT-5 Synthesis Integration**: Independent executive analysis agent for human-readable report synthesis

### Configuration System
- **Complete Agent Control**: `config/agents.yaml` with per-agent model, temperature, max_tokens, rate_limit_ms, and custom prompts
- **Agent Management CLI**: `scripts/manage_agents.py` for interactive configuration, cloning, and topology visualization  
- **Pydantic Settings**: `config/settings.py` with environment variable support and validation
- **Dynamic Agent Factory**: `sefas/agents/factory.py` creates agents from configuration with graceful fallbacks

## Development Commands

### Setup
```bash
# Install dependencies
make install

# Configure environment
cp .env.example .env
# Edit .env with your OpenAI and LangSmith API keys
```

### Running the System
```bash
# Basic task execution (uses default demo task)
make run

# Direct task execution (automatic 'run' command insertion)
python scripts/run_experiment.py "Your task here" --max-hops 15 --verbose

# Explicit run command
python scripts/run_experiment.py run "Your task here" --experiment-name "test-run"

# Batch experiments
python scripts/run_experiment.py batch tasks.json --output-dir results/
```

### Development Workflow
```bash
# Run tests
make test                    # All tests
make test-unit              # Unit tests only
make test-integration       # Integration tests only
pytest tests/unit/test_agents/test_orchestrator.py  # Single test file

# Comprehensive testing
python test_agents.py       # Full system test suite with 5 challenge types
python test_improvements.py # Test enhanced belief propagation, validation, and redundancy

# Agent Management
python scripts/manage_agents.py list              # View all 17 agents
python scripts/manage_agents.py show orchestrator # Detailed agent configuration
python scripts/manage_agents.py edit proposer_alpha # Interactive configuration editing
python scripts/manage_agents.py topology          # Network visualization
python scripts/manage_agents.py quick-config      # Batch modifications

# GPT-5 Synthesis Agent (independent executive analysis)
python scripts/gpt5_synthesis_agent.py --auto <task_id>  # Auto-detect reports by task ID
python scripts/gpt5_synthesis_agent.py --json report.json --html report.html --md report.md  # Manual paths

# Evolution System (currently disabled - enable in config/settings.py)
# Set evolution_enabled: bool = True in settings.py to activate

# Code quality
make lint                   # ruff + mypy
make format                 # ruff format + black

# Cleanup
make clean                  # Remove cache files
```

### Troubleshooting Commands
```bash
# Test with verbose output and enhanced reporting
python scripts/run_experiment.py "Your task" --verbose

# Environment variable debugging (common API key issues)
unset OPENAI_API_KEY && python scripts/run_experiment.py "test task"

# Debug validation layer issues (FULLY RESOLVED ✅)
python test_confidence_calibration.py  # Test belief propagation improvements
python scripts/run_experiment.py "test validation" --verbose  # Verify 100% validation pass rate

# GPT-5 Synthesis Troubleshooting
ls data/reports/gpt5_executive_report_*.txt  # Check generated synthesis reports

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keef75/SEFAS-GPT5](https://github.com/keef75/SEFAS-GPT5) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
