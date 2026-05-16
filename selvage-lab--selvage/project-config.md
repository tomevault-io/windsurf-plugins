---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Testing

```bash
# Run unit and integration tests
pytest tests/

# Run E2E tests (requires testcontainers and docker)
pytest e2e/

# Build fresh TestPyPI Docker image (ensures latest selvage version)
./scripts/build_testpypi_image.sh

# Or rebuild manually with cache bypass
docker build --no-cache --build-arg CACHEBUST=$(date +%s) -t selvage-testpypi:latest -f e2e/dockerfiles/testpypi/Dockerfile .

# Run LLM evaluation tests
pytest llm_eval/

# Run tests with coverage
pytest tests/ --cov

# Run specific test patterns
pytest tests/test_llm_gateway.py -v

# Run specific test files
pytest tests/test_config_env_vars.py::test_config_language -v
pytest tests/test_cli_flags.py -v

# Run tests with parallel execution
pytest tests/ -n auto

# Run tests with timeout
pytest tests/ --timeout=300
```

### Package Management

```bash
# Install for development
pip install -e .[dev]

# Install with E2E test dependencies
pip install -e .[dev,e2e]

# Build package
python -m build

# Upload to PyPI (requires twine)
twine upload dist/*
```

### Running the Application

```bash
# Basic code review
selvage review

# Review staged changes
selvage review --staged

# Review with specific model
selvage review --model claude-sonnet-4

# Launch UI to view results
selvage view

# Set API keys (환경변수)
export OPENAI_API_KEY=your_openai_key
export ANTHROPIC_API_KEY=your_claude_key
export GEMINI_API_KEY=your_gemini_key
export OPENROUTER_API_KEY=your_openrouter_key

# Configure default model and language
selvage config model claude-sonnet-4
selvage config language ko  # Set Korean language
selvage config debug-mode on

# View current configuration
selvage config list
```

## Architecture Overview

Selvage is an LLM-based code review tool with a modular architecture:

### Core Components

**CLI Interface** (`selvage/cli.py`)

- Main entry point using Click framework
- Handles command parsing and user interaction
- Manages API key setup and configuration

**LLM Gateway System** (`selvage/src/llm_gateway/`)

- `gateway_factory.py`: Factory pattern for creating LLM clients
- `base_gateway.py`: Abstract base class for all LLM providers
- Provider-specific gateways: `openai_gateway.py`, `claude_gateway.py`, `google_gateway.py`, `openrouter_gateway.py`
- Handles API communication, token counting, and cost estimation

**Diff Processing** (`selvage/src/diff_parser/`)

- `parser.py`: Git diff parsing and processing
- `models/`: Data models for diff representation (DiffResult, FileDiff, Hunk)
- Converts git diff output into structured data for LLM analysis

**Prompt System** (`selvage/src/utils/prompts/`)

- `prompt_generator.py`: Creates structured prompts for LLM review
- `models/`: Prompt data models (ReviewPromptWithFileContent, SystemPrompt, UserPromptWithFileContent)
- **Context Extraction System**: Intelligent file context extraction with three types:
  - `SMART_CONTEXT`: AST-based tree-sitter analysis for related code blocks
  - `FALLBACK_CONTEXT`: Text-based pattern matching when AST analysis fails
  - `FULL_CONTEXT`: Complete file content for new/rewritten files

**Configuration Management** (`selvage/src/config.py`)

- Platform-specific config directory handling
- API key management (environment variables take precedence)
- User preferences storage (default model, debug mode, etc.)

### Data Flow

1. **Git Diff Extraction**: `GitDiffUtility` extracts changes from repository
2. **Diff Parsing**: `DiffParser` converts raw diff into structured `DiffResult`
3. **Prompt Generation**: `PromptGenerator` creates context-aware prompts
4. **LLM Review**: Gateway sends prompts to configured AI model
5. **Result Processing**: Response is formatted and saved as review log
6. **UI Display**: Streamlit interface shows results with markdown/JSON views

### Key Design Patterns

- **Factory Pattern**: `GatewayFactory` for LLM provider instantiation
- **Strategy Pattern**: Different diff modes (staged, unstaged, target branch/commit)
- **Template Method**: Base gateway class with provider-specific implementations
- **Configuration**: Environment variables override file-based settings

### Model Support

The tool supports multiple AI providers:

- **OpenAI**: GPT-5.2 Codex
- **Anthropic**: Claude Opus 4.6, Claude Sonnet 4.5
- **Google**: Gemini 3 Pro, Gemini 3 Flash
- **OpenRouter**: MiniMax M2.5, GLM-5, Qwen3 Coder, Kimi K2.5, DeepSeek R1/V3 and more

Model configuration is centralized in `selvage/resources/models.yml` with provider-specific settings.

## Testing Strategy

- **Unit Tests** (`tests/`): Component-level testing
- **E2E Tests** (`e2e/`): Full workflow testing with Docker containers
- **LLM Evaluation** (`llm_eval/`): AI model response quality assessment using DeepEval

## Git Workflow Integration

Selvage supports various Git diff modes for flexible code review:

```bash
# Review unstaged changes (default)
selvage review

# Review staged changes only
selvage review --staged

# Review changes against specific branch
selvage review --target-branch main
selvage review --target-branch develop

# Review changes from specific commit to HEAD

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [selvage-lab/selvage](https://github.com/selvage-lab/selvage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
