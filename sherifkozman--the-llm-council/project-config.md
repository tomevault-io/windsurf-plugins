---
trigger: always_on
description: Multi-LLM Council Framework (v0.6.0) that orchestrates multiple LLM backends for adversarial debate, cross-validation, and structured decision-making. Published as `the-llm-council` on PyPI.
---

# The LLM Council - Development Guide

## Project Overview

Multi-LLM Council Framework (v0.6.0) that orchestrates multiple LLM backends for adversarial debate, cross-validation, and structured decision-making. Published as `the-llm-council` on PyPI.

- **Python**: >=3.10 (tested 3.10, 3.11, 3.12)
- **Build**: hatchling
- **CLI**: typer + rich
- **Source**: `src/llm_council/`
- **Tests**: `tests/` (pytest + pytest-asyncio)
- **Linting**: ruff, mypy (strict)

## Quick Reference

```bash
council run <subagent> "<task>"       # Run a council task
council run drafter --mode impl "..." # With specific mode
council doctor                        # Check provider health
council config --show                 # View configuration
council version                       # Show version
```

## Architecture

```
Council (facade) → Orchestrator → Provider Adapters
                        ↓
                  3-phase flow:
                  1. Parallel drafts (asyncio.gather)
                  2. Adversarial critique
                  3. Schema-validated synthesis (with retry)
```

### Source Layout

```
src/llm_council/
├── cli/main.py          # CLI entry point (typer app)
├── council.py           # Council facade class
├── config/models.py     # Model packs, env var config
├── engine/
│   ├── orchestrator.py  # Core 3-phase orchestration
│   ├── degradation.py   # Graceful degradation policy
│   └── health.py        # Provider health checks
├── protocol/types.py    # Pydantic types (CouncilConfig, etc.)
├── providers/
│   ├── base.py          # ProviderAdapter interface
│   ├── registry.py      # Provider entry point registry
│   ├── openrouter.py    # OpenRouter (recommended)
│   ├── anthropic.py     # Direct Anthropic API
│   ├── openai.py        # Direct OpenAI API
│   ├── google.py        # Direct Google API
│   ├── vertex.py        # Vertex AI (Gemini + Claude)
│   └── cli/             # CLI-based providers
│       ├── codex.py     # OpenAI Codex CLI
│       └── gemini.py    # Gemini CLI
├── schemas/             # JSON schemas per subagent
├── storage/
│   ├── artifacts.py     # Artifact store (aiosqlite)
│   └── summarize.py     # Tiered summarization
├── subagents/           # YAML configs per subagent
├── registry/            # Base agent, tool registry
└── validation/          # Output validation
```

## Subagents

### Core Agents (v0.5.0+)

| Subagent | Use For | Modes |
|----------|---------|-------|
| `drafter` | Generate code, designs, tests | `impl`, `arch`, `test` |
| `critic` | Code review, security analysis | `review`, `security` |
| `synthesizer` | Merge and finalize outputs | - |
| `researcher` | Technical/market research | - |
| `planner` | Roadmaps, build-vs-buy | `plan`, `assess` |
| `router` | Classify and route tasks | - |

### Deprecated Aliases (Backwards Compatible)

| Old Name | Maps To | Removed In |
|----------|---------|------------|
| `implementer` | `drafter --mode impl` | v1.0 |
| `architect` | `drafter --mode arch` | v1.0 |
| `test-designer` | `drafter --mode test` | v1.0 |
| `reviewer` | `critic --mode review` | v1.0 |
| `red-team` | `critic --mode security` | v1.0 |
| `assessor` | `planner --mode assess` | v1.0 |
| `shipper` | `synthesizer` | v1.0 |

### Subagent Configs

YAML files in `src/llm_council/subagents/` define per-agent:
- Mode-specific model packs and schemas
- Reasoning budgets (OpenAI effort, Anthropic budget_tokens, Gemini thinking_level)
- Provider preferences (preferred, fallback, exclude)
- Model overrides per provider
- System and mode-specific prompts

## CLI Reference

### `council run`

```bash
council run <subagent> "<task>" [OPTIONS]
```

| Option | Description |
|--------|-------------|
| `--mode` | Agent mode (impl/arch/test, review/security, plan/assess) |
| `--providers, -p` | Comma-separated provider list |
| `--models, -m` | Comma-separated OpenRouter model IDs for multi-model council |
| `--timeout, -t` | Request timeout in seconds |
| `--temperature` | Model temperature (0.0-2.0) |
| `--max-tokens` | Max output tokens |
| `--files, -f` | File paths as context (repeatable or comma-separated; 50KB/file, 200KB total) |
| `--input, -i` | Read task from file (use `-` for stdin) |
| `--output, -o` | Write output to file |
| `--context, --system` | Additional system context/instructions |
| `--schema` | Custom output schema JSON file |
| `--no-artifacts` | Disable artifact storage |
| `--json` | Output structured JSON |
| `--verbose, -v` | Show detailed output with metrics |
| `--dry-run` | Show what would run without executing |

### `council doctor`

```bash
council doctor [--json] [--provider NAME]
```

### `council config`

```bash
council config --show          # Show current configuration
council config --init          # Create default config at ~/.config/llm-council/config.yaml
council config --path          # Show config file path
council config --validate      # Validate config file
council config --get KEY       # Get value by dot notation
council config --set KEY VALUE # Set value
council config --edit          # Open in $EDITOR
```

### Global Options

```bash
council --version/-V    # Show version

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sherifkozman/the-llm-council](https://github.com/sherifkozman/the-llm-council) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
