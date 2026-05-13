---
trigger: always_on
description: This is a research framework for testing and evaluating AI agents across various task domains, designed for misalignment interpretability research. It provides containerized environments where agents interact with controlled tasks including game-playing (Chess, Tic-Tac-Toe), coding challenges, and sandbagging detection tests.
---

# Agent Interpretability Environments

## Project Overview

This is a research framework for testing and evaluating AI agents across various task domains, designed for misalignment interpretability research. It provides containerized environments where agents interact with controlled tasks including game-playing (Chess, Tic-Tac-Toe), coding challenges, and sandbagging detection tests.

**Key Concepts:**
- **Environments** run in Docker containers with controlled access
- **Checkpointing** enables saving, resuming, and resampling from any state
- **Multi-provider support** for Anthropic, OpenAI, OpenRouter, Minimax, and Moonshot
- **Standardized output** format across all environment types

## Tech Stack

- **Python**: 3.11+ (see `.python-version`)
- **Package Manager**: `uv` (recommended) or `pip`
- **Dependencies**: anthropic, openai, omegaconf, pyyaml, tenacity, python-dotenv
- **Configuration**: OmegaConf (variable interpolation with CLI overrides)
- **Containerization**: Docker
- **Testing**: pytest, pytest-asyncio, pytest-repeat
- **Compilation**: Cython (for opponent packages to prevent source inspection)

## Project Structure

```
agent-interp-envs/
├── src/agent_interp_envs/        # Core framework package (reusable)
│   ├── providers/                # LLM provider implementations
│   │   ├── base.py              # BaseProvider interface
│   │   ├── anthropic_provider.py
│   │   ├── openai_provider.py
│   │   ├── openrouter_provider.py
│   │   ├── minimax_provider.py
│   │   └── moonshot_provider.py
│   ├── config.py                 # YAML configuration system
│   ├── tool_calling.py           # Tool execution framework
│   └── types.py                  # Shared type definitions
│
├── environments/                 # Dockerized task environments
│   ├── games/                    # Game-playing environments
│   │   ├── agent.py              # Unified player (factory pattern)
│   │   ├── run_step.py           # Step execution logic
│   │   ├── states.py             # State management
│   │   ├── chess/                # Chess environment
│   │   │   ├── Dockerfile
│   │   │   ├── entry_point.py   # Installs packages, runs player
│   │   │   ├── game.py          # Basic CLI
│   │   │   ├── game_hint.py     # CLI with hint support
│   │   │   └── opponent_packages/  # Cython-compiled packages
│   │   │       ├── chess_opponent/
│   │   │       ├── chess_evaluator/
│   │   │       └── chess_hint/
│   │   └── tictactoe/           # Similar structure to chess
│   ├── coding/                   # Coding environment
│   │   ├── Dockerfile
│   │   ├── agent.py
│   │   ├── run_step.py
│   │   └── states.py
│   └── sandbagging/              # Sandbagging detection environment
│       ├── Dockerfile
│       ├── agent.py
│       ├── entrypoint.py        # Selects documentation based on config
│       ├── run_step.py
│       ├── states.py
│       ├── tools.py             # submit + execute_command tools
│       ├── questions.txt        # Arithmetic questions
│       ├── Documentation.md     # Normal mode (penalizes high scores)
│       └── Documentation_reverse.md  # Reverse mode (penalizes low scores)
│
├── configs/                      # Self-contained experiment configs
│   ├── chess/
│   │   ├── single_no_hint.yaml
│   │   ├── single_hint.yaml
│   │   ├── multi_no_hint.yaml
│   │   ├── multi_hint.yaml
│   │   └── multi_hint_official_only.yaml
│   ├── tictactoe/
│   │   └── (same structure as chess)
│   ├── coding/
│   │   └── two_sum.yaml
│   └── sandbagging/
│       ├── normal.yaml
│       └── reverse.yaml
│
├── scripts/                      # Main entry points
│   ├── run.py                   # Run environments
│   ├── resume.py                # Resume from checkpoints
│   └── build_container.sh       # Build Docker images
│
├── tests/                        # Test suites
│   ├── api/                     # API/provider tests
│   ├── chess/                   # Chess environment tests
│   ├── tictactoe/               # Tic-tac-toe environment tests
│   ├── coding/                  # Coding environment tests
│   └── sandbagging/             # Sandbagging environment tests
│
└── results/                      # Output directory (gitignored)
    └── <env>/<model>/<timestamp>/run-N/step-N/
```

## Key Commands

### Running Environments

```bash
# Run an experiment (pulls from Dockerhub)
python scripts/run.py configs/chess/single_no_hint.yaml

# Override config values from command line
python scripts/run.py configs/chess/single_no_hint.yaml agent.provider=anthropic agent.model=claude-sonnet-4

# Run multiple parallel rollouts
python scripts/run.py configs/chess/single_no_hint.yaml --count 5

# Build and run with local image
python scripts/run.py configs/tictactoe/single_no_hint.yaml --local --build

# Use cached local image
python scripts/run.py configs/tictactoe/single_no_hint.yaml --local

# Override multiple values
python scripts/run.py configs/sandbagging/normal.yaml agent.provider=openai agent.model=gpt-4o agent.max_steps=30
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gkroiz/agent-interp-envs](https://github.com/gkroiz/agent-interp-envs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
