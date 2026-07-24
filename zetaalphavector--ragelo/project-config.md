---
trigger: always_on
description: RAGElo is a Python library and CLI for evaluating RAG (Retrieval-Augmented Generation) agents using Elo-based tournament ranking. It evaluates both document retrieval quality and answer quality across multiple RAG pipeline variations.
---

# Copilot Instructions — RAGElo

## Project Overview

RAGElo is a Python library and CLI for evaluating RAG (Retrieval-Augmented Generation) agents using Elo-based tournament ranking. It evaluates both document retrieval quality and answer quality across multiple RAG pipeline variations.

## Commands

```bash
# Install for development
uv pip install -e '.[dev]'

# Run all tests (OpenAI integration tests are skipped by default)
pytest tests/

# Run with OpenAI integration tests (requires OPENAI_API_KEY)
pytest tests/ --runopenai

# Run a single test file or specific test
pytest tests/unit/test_experiment.py -v
pytest tests/unit/test_experiment.py::TestExperiment::test_method -v

# Lint and format (ruff rules: E, F, I; line-length: 119)
ruff check ragelo/
ruff format ragelo/

# Type checking (uses pydantic.mypy plugin)
mypy ragelo/

# Pre-commit hooks (ruff lint + format)
pre-commit run --all-files
```

## Architecture

### Factory + Registry Pattern

All major components use decorator-based factory registration. Enum types in `ragelo/types/types.py` define valid component names (`RetrievalEvaluatorTypes`, `AnswerEvaluatorTypes`, `LLMProviderTypes`, `AgentRankerTypes`).

```python
# Registration (on class definition)
@RetrievalEvaluatorFactory.register(RetrievalEvaluatorTypes.REASONER)
class ReasonerEvaluator(BaseRetrievalEvaluator): ...

# Instantiation (via factory functions)
evaluator = get_retrieval_evaluator("reasoner", llm_provider=provider)
provider = get_llm_provider("openai", api_key="...")
ranker = get_agent_ranker("elo")
```

When adding a new evaluator, LLM provider, or ranker:
1. Add the name to the corresponding `StrEnum` in `ragelo/types/types.py`.
2. Create a config class in `ragelo/types/configurations/`.
3. Implement the class and decorate it with the factory's `@register`.
4. The class is now accessible via the factory function and CLI.

### Evaluator Hierarchy

```
BaseEvaluator (ragelo/evaluators/base_evaluator.py) — async-first, abstract
├── BaseRetrievalEvaluator — evaluates document relevance (Query + Document → score)
│   Implementations: Reasoner, RDNAM, DomainExpert, FewShot, CustomPrompt
└── BaseAnswerEvaluator — evaluates answer quality (Query + AgentAnswer → score/winner)
    Implementations: Pairwise, ChatPairwise, CustomPairwise, DomainExpert, CustomPrompt
```

All evaluations are **async**: `evaluate_async()` is the core abstract method. `evaluate_experiment()` orchestrates bounded concurrent execution via `asyncio.wait()` with configurable `n_processes`.

### LLM Providers

`BaseLLMProvider` defines the interface: `call_async(input, response_schema) → LLMResponseType[T]`. Implementations: `OpenAIProvider` (structured output via Responses API), `OllamaProvider`, `InstructorProvider`. Providers return `LLMResponseType[T]` with `raw_answer` and `parsed_answer`.

`InstructorProvider` is registered as `"instructor"` and uses the [`instructor`](https://github.com/jxnl/instructor) library to support multiple backends (Anthropic, OpenAI, Mistral, Cohere) through a unified interface. It is an **optional dependency** — requires `pip install 'ragelo[instructor]'` plus the relevant provider SDK. Instantiation raises `ImportError` with a helpful message if instructor is not installed, so `import ragelo` always works regardless.

Supported backends and their call path:
| `provider=` | SDK package | API path used |
|---|---|---|
| `"openai"` | `openai` | `chat.completions.create` |
| `"anthropic"` | `anthropic` | `messages.create` |
| `"mistral"` | `mistralai` | `chat.completions.create` |
| `"cohere"` | `cohere` | `chat.completions.create` |

**Conditional import pattern** — `instructor_client.py` wraps `import instructor` in a module-level `try/except` and checks `_INSTRUCTOR_AVAILABLE` in `__init__`. This is the standard Python pattern for optional dependencies and is a necessary exception to the "no conditional imports" rule. The `__init__.py` can import `InstructorProvider` unconditionally since the module-level import always succeeds (just sets a flag).

### Data Model

- **Core types** (`ragelo/types/evaluables.py`): `Query`, `Document`, `AgentAnswer`, `PairwiseGame`
- **Results** (`ragelo/types/results.py`): `EvaluatorResult`, `RetrievalEvaluatorResult`, `AnswerEvaluatorResult`, `PairwiseGameEvaluatorResult`
- **LLM answer schemas** (`ragelo/types/answer_formats.py`): Pydantic models for structured LLM outputs
- **Configs** (`ragelo/types/configurations/`): Pydantic config classes per component, resolved via `get_config_class()` introspection

### Experiment Orchestration

`Experiment` (`ragelo/types/experiment.py`) is the central orchestrator: loads queries/documents/answers from CSV, manages evaluation state with caching to avoid redundant LLM calls, and persists to JSON/JSONL.

### Prompt Templating

Jinja2 templates for all LLM prompts. Available context variables: `{{ query.query }}`, `{{ document.text }}`, `{{ answer.text }}`, `{{ game.agent_a_answer.text }}`. Extra CSV columns become metadata: `{{ query.metadata.column_name }}`.

### CLI


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zetaalphavector/RAGElo](https://github.com/zetaalphavector/RAGElo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
