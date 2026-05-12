---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Install dependencies
uv sync

# Install with dev dependencies
uv sync --all-extras

# Run tests
uv run pytest

# Run single test file
uv run pytest tests/test_file.py

# Run linting
uv run ruff check .

# Format code (REQUIRED before committing)
uv run ruff format .

# Run type checking
uv run mypy src/cv_warlock

# Start web UI
uv run warlock-streamlit

# CLI usage
uv run cv-warlock tailor my_cv.md job.txt -o tailored_cv.md
uv run cv-warlock analyze my_cv.md job.txt

# Test API keys
uv run python scripts/test_api_keys.py
uv run python scripts/test_api_keys.py anthropic  # Test specific provider
```

## Testing with LangSmith

**Important**: Always use LangSmith for tracing when testing workflows. LangSmith is configured via `.env.local` with:

- `LANGCHAIN_TRACING_V2=true`
- `LANGCHAIN_PROJECT=cv-warlock`
- `LANGCHAIN_API_KEY=<your-key>`

View traces at: <https://smith.langchain.com>

### Test Output Location

**IMPORTANT**: All test output files MUST be saved in the `test_results/` folder, which is gitignored. Never create test output files in the project root or other directories.

### Test Output Naming Convention

**IMPORTANT**: All test output files MUST follow this naming format:

```text
test_results/YYYY-MM-DD_HH-MM_modelname.md
```

Examples:

- `test_results/2026-01-17_14-30_claude-sonnet-4-5.md`
- `test_results/2026-01-17_09-15_gpt-5-2.md`
- `test_results/2026-01-17_16-45_gemini-3-pro.md`

Example test commands with tracing (model is auto-selected via Dual-Model Strategy):

```bash
# Test with Anthropic (default)
uv run cv-warlock tailor examples/sample_cv.md examples/sample_job_posting.md -o test_results/2026-01-17_14-30_anthropic.md -p anthropic -v

# Test with OpenAI
uv run cv-warlock tailor examples/sample_cv.md examples/sample_job_posting.md -o test_results/2026-01-17_14-30_openai.md -p openai -v

# Test with Google
uv run cv-warlock tailor examples/sample_cv.md examples/sample_job_posting.md -o test_results/2026-01-17_14-30_google.md -p google -v
```

### Test Data Guidelines

**CRITICAL**: Always use REAL sample data from `examples/` directory in tests, NOT mock/fake data:

- `examples/sample_cv.md` - Real-world CV with actual experiences, skills, education
- `examples/sample_job_posting.md` - Real-world job posting with actual requirements

**Why this matters:**

- Mock data often has unrealistic structures that pass tests but fail in production
- Real examples validate the full parsing and extraction pipeline
- Tests catch edge cases that only appear with real-world formatting
- RLM orchestrator requires realistic document structure to function correctly

**In pytest fixtures**, load real data:

```python
from pathlib import Path

PROJECT_ROOT = Path(__file__).parent.parent

@pytest.fixture
def sample_cv_text():
    """Load REAL sample CV from examples/sample_cv.md."""
    cv_path = PROJECT_ROOT / "examples" / "sample_cv.md"
    return cv_path.read_text(encoding="utf-8")

@pytest.fixture
def sample_job_text():
    """Load REAL sample job posting from examples/sample_job_posting.md."""
    job_path = PROJECT_ROOT / "examples" / "sample_job_posting.md"
    return job_path.read_text(encoding="utf-8")
```

## Architecture

### LangGraph Workflow
The core processing is a LangGraph `StateGraph` defined in `graph/workflow.py`:

```
validate_inputs → extract_cv → extract_job → analyze_match → create_plan
    → tailor_summary → tailor_experiences → tailor_skills → assemble_cv
```

- **State**: `CVWarlockState` (TypedDict in `models/state.py`) flows through all nodes
- **Nodes**: Created in `graph/nodes.py` via `create_nodes(llm_provider, use_cot)` factory
- **Edges**: Conditional edges handle error paths (skip to END if errors exist)

### LLM Provider Abstraction
- Base class `LLMProvider` in `llm/base.py` with `get_chat_model()` and `get_extraction_model()`
- Implementations: `AnthropicProvider`, `OpenAIProvider`, `GoogleProvider`
- Factory function `get_llm_provider(provider, model, api_key)` for instantiation
- Uses LangChain's `with_structured_output()` for Pydantic model extraction

### Chain-of-Thought (CoT) Mode
When `use_cot=True` (default), each tailoring step follows REASON → GENERATE → CRITIQUE → REFINE:
- `CVTailor` in `processors/tailor.py` has both direct and CoT methods
- CoT outputs stored in `*_reasoning_result` fields for transparency
- `GenerationContext` in `models/reasoning.py` maintains consistency across sections

### RLM (Recursive Language Model) Mode

When `use_rlm=True`, enables recursive orchestration for handling large documents:

- `RLMOrchestrator` in `rlm/orchestrator.py` manages the REPL-based analysis loop
- Root model writes Python code to explore CV/job data, calls sub-models for deep analysis
- RLM nodes in `graph/rlm_nodes.py` wrap extraction steps with RLM capability
- Falls back to standard extraction if RLM output isn't parseable as structured data
- Configured via `RLMConfig` with size thresholds, iteration limits, timeouts

### Key Models

- `CVData` (`models/cv.py`): Parsed CV structure with experiences, skills, education

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WSzP/cv-warlock](https://github.com/WSzP/cv-warlock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
