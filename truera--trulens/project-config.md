---
trigger: always_on
description: - Install deps: `poetry install`
---

# AGENTS.md

## Setup commands

- Install deps: `poetry install`
- Install with dev tools: `poetry install --with dev`
- Install with all optional packages: `poetry install --with dev,apps,providers`
- Install pre-commit hooks: `pre-commit install`
- Start documentation server: `make docs-serve`

## Code style

- Line length: 80 characters
- Formatter/linter: `ruff` (run `make format` and `make lint`)
- Google-style docstrings
- Import modules, not classes directly:
  ```python
  # ✓ Do this
  from trulens.schema import record as record_schema
  from trulens.providers.openai import provider as openai_provider

  # ✗ Not this
  from trulens.schema.record import Record
  ```
- Standard module rename patterns:
  ```python
  from trulens.schema import X as X_schema
  from trulens.utils import X as X_utils
  from trulens.providers.X import provider as X_provider
  from trulens.apps.X import Y as Y_app
  from trulens.core import X as core_X
  from trulens.core.database import base as core_db
  from trulens.feedback.templates import rag as templates_rag
  from trulens.feedback.templates import base as templates_base
  ```
- Use `TYPE_CHECKING` blocks for type-only imports
- Use `from __future__ import annotations` for forward references
- Call `model_rebuild()` after Pydantic models with forward refs

## Testing instructions

- Run all unit tests: `make test-unit`
- Run single test file: `TEST_OPTIONAL=true poetry run pytest tests/unit/test_file.py -v`
- Run specific test: `TEST_OPTIONAL=true poetry run pytest tests/unit/test_file.py::TestClass::test_method`
- OTEL tests require isolation (uses pytest-xdist): `TEST_OPTIONAL=1 poetry run pytest tests/unit/test_otel*.py -n auto --dist=loadscope`
- Regenerate golden files: `WRITE_GOLDEN=1 TEST_OPTIONAL=1 poetry run pytest <test_path>`

### Test markers

- `@pytest.mark.optional` - requires optional dependencies
- `@pytest.mark.snowflake` - requires Snowflake credentials
- `@pytest.mark.huggingface` - requires HuggingFace access

Enable optional tests: `TEST_OPTIONAL=true`

## Build commands

- Format code: `make format`
- Lint code: `make lint`
- Build all packages: `make build`
- Build docs: `make docs`
- Update poetry locks: `make lock`
- Generate coverage: `make coverage`

## Project structure

```
src/
├── core/           # trulens-core: Core abstractions, session, database
├── feedback/       # trulens-feedback: Feedback function implementations
│   ├── templates/  # Prompt template classes organized by domain:
│   │   ├── base.py     # FeedbackTemplate base class, shared scaffolding
│   │   ├── rag.py      # RAG evals (Groundedness, ContextRelevance, …)
│   │   ├── safety.py   # Moderation (Harmfulness, Toxicity, …)
│   │   ├── quality.py  # Text quality (Coherence, Sentiment, …)
│   │   └── agent.py    # Agentic evals (ToolSelection, PlanAdherence, …)
│   ├── llm_provider.py # LLMProvider base — orchestrates template → LLM calls
│   ├── v2/             # Backward-compat shim (re-exports from templates/)
│   └── prompts.py      # Backward-compat shim (re-exports from templates/)
├── dashboard/      # trulens-dashboard: Streamlit UI + React components
├── apps/           # App integrations (langchain, langgraph, llamaindex)
├── providers/      # LLM providers (openai, bedrock, cortex, huggingface, litellm)
├── connectors/     # Database connectors (snowflake)
└── otel/semconv/   # OpenTelemetry semantic conventions
```

## Key patterns

### TruSession (main entry point)
```python
from trulens.core import TruSession
session = TruSession()  # Default SQLite
session = TruSession(database_url="postgresql://...")
```

### App wrappers
```python
from trulens.apps.langchain import TruChain
tru_app = TruChain(chain, app_name="MyApp", app_version="v1", feedbacks=[...])
with tru_app as recording:
    result = chain.invoke("query")
```

### OTEL instrumentation

Basic instrumentation - captures function args and return as span attributes:
```python
from trulens.core.otel.instrument import instrument

@instrument()
def my_function():
    pass  # Automatically traced
```

#### Span types

Use `span_type` to categorize spans for semantic meaning:
```python
from trulens.core.otel.instrument import instrument
from trulens.otel.semconv.trace import SpanAttributes

@instrument(span_type=SpanAttributes.SpanType.RETRIEVAL)
def retrieve(self, query: str) -> list:
    pass

@instrument(span_type=SpanAttributes.SpanType.GENERATION)
def generate(self, prompt: str) -> str:
    pass
```

Available span types: `RETRIEVAL`, `GENERATION`, `RERANKING`, `TOOL`, `AGENT`, `WORKFLOW`, `GRAPH_NODE`, `GRAPH_TASK`, `MCP`, `GUARDRAIL`, `RECORD_ROOT`, `EVAL_ROOT`, `UNKNOWN`

#### Custom span attributes

Map function args/return to semantic attributes:
```python
@instrument(
    span_type=SpanAttributes.SpanType.RETRIEVAL,
    attributes={
        SpanAttributes.RETRIEVAL.QUERY_TEXT: "query",      # maps "query" arg
        SpanAttributes.RETRIEVAL.RETRIEVED_CONTEXTS: "return",  # maps return value
    },
)
def retrieve(self, query: str) -> list:
    return ["context1", "context2"]
```

Common attribute namespaces:
- `SpanAttributes.RETRIEVAL` - `QUERY_TEXT`, `RETRIEVED_CONTEXTS`, `NUM_CONTEXTS`
- `SpanAttributes.RECORD_ROOT` - `INPUT`, `OUTPUT`, `ERROR`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [truera/trulens](https://github.com/truera/trulens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
