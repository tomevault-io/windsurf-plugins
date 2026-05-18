---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Essential Commands

```bash
# Setup & Development
uv pip install -e ".[dev]"                              # Install dependencies
uv run mcp-server-mas-sequential-thinking               # Run server
uv run ruff check . --fix && uv run ruff format . && uv run mypy .  # Code quality

# Testing Framework (using Makefile)
make test                                               # Run all tests with coverage + quality checks
make test-unit                                          # Run unit tests only
make test-integration                                   # Run integration tests only
make test-security                                      # Run security tests only
make test-config                                        # Run configuration tests
make test-fast                                          # Fast run without coverage
make test-parallel                                      # Run tests in parallel
make test-coverage                                      # Generate HTML coverage report
make check-all                                          # Run all quality checks
make help                                               # Show all available commands

# Direct Pytest Commands
uv run pytest tests/ -v                                 # Run all tests
uv run pytest -m unit                                   # Run tests with specific marker
uv run pytest --no-cov                                  # Skip coverage

# Available pytest markers: unit, integration, security, config, slow, asyncio
# asyncio_mode = auto (no explicit @pytest.mark.asyncio needed)
# Coverage threshold: --cov-fail-under=70

# Debugging & Monitoring
tail -f ~/.sequential_thinking/logs/sequential_thinking.log          # Live logs
npx @modelcontextprotocol/inspector uv run mcp-server-mas-sequential-thinking  # Test server
```

## Project Overview

**AI-powered Multi-Thinking implementation** using Agno 2.5.9+ framework via MCP. Processes thoughts through six cognitive perspectives (Factual, Emotional, Critical, Optimistic, Creative, Synthesis) with intelligent complexity analysis determining execution strategy (Single/Double/Triple/Full sequences).

**Core Flow:** External LLM -> `sequentialthinking` tool -> AI complexity analysis -> Multi-Thinking workflow -> Individual hat agents -> Synthesis

## Configuration

**Required Environment Variables:**
```bash
LLM_PROVIDER=deepseek                                    # Provider (deepseek, groq, openrouter, ollama, github, anthropic)
DEEPSEEK_API_KEY=your_key                               # Provider API key
DEEPSEEK_ENHANCED_MODEL_ID=deepseek-chat                # Synthesis model
DEEPSEEK_STANDARD_MODEL_ID=deepseek-chat                # Individual hats model
EXA_API_KEY=your_key                                    # Optional: Research capabilities
TEAM_MODE=standard                                      # Team mode (standard/broadcast, route, coordinate)
```

**Model Strategy:**
- **Enhanced Model**: Blue Hat (synthesis) for complex integration
- **Standard Model**: Individual hats (White, Red, Black, Yellow, Green) for focused thinking
- **AI Selection**: System automatically chooses model based on hat type and complexity

## Key Architecture

**Entry Point:** `src/mcp_server_mas_sequential_thinking/main.py`

**Core Services:**
- `ThoughtProcessor`: Main orchestrator with dependency injection
- `WorkflowExecutor`: Manages Multi-Thinking workflow execution
- `AIComplexityAnalyzer`: AI-driven complexity assessment (replaces rule-based patterns)
- `MultiThinkingSequentialProcessor`: Executes chosen thinking sequence

**Processing Strategies (AI-Determined):**
1. **Single Hat**: Simple focused thinking
2. **Double Hat**: Two-step sequences (e.g., Optimistic->Critical)
3. **Triple Hat**: Core philosophical thinking (Factual->Creative->Synthesis)
4. **Full Sequence**: Complete Multi-Thinking with Blue Hat orchestration

## Critical Development Patterns

**Dependency Injection:** Manual constructor injection, Protocol-based interfaces in `core/types.py`

**Import Safety:** Avoid circular dependencies:
```python
from typing import TYPE_CHECKING
if TYPE_CHECKING:
    from module import Class
```

**Thread Safety:** Global state uses async locks:
```python
_processor_lock = asyncio.Lock()
async with _processor_lock:
    # Safe initialization
```

**Error Handling:** Use `ThoughtProcessingError` hierarchy, include `ProcessingMetadata` for debugging

**Parallel Processing:** Non-synthesis agents use `asyncio.gather` for simultaneous execution

**TeamMode (secondary path):** `ProcessingOrchestrator._get_team()` constructs Agno Teams with `TeamMode` based on `TEAM_MODE` env var. Primary path (`MultiThinkingSequentialProcessor`) uses manual `asyncio.gather` and is unaffected. Mapping: standard/broadcast -> `TeamMode.broadcast`, coordinate -> `TeamMode.coordinate`, route -> `TeamMode.route`.

---

## Agno 2.5.9+ Modern Patterns

### Typed State Management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FradSer/mcp-server-mas-sequential-thinking](https://github.com/FradSer/mcp-server-mas-sequential-thinking) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
