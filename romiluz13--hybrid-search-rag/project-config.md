---
trigger: always_on
description: This guide helps AI agents effectively work with the HybridRAG codebase.
---

# HybridRAG - AI Agent Development Guide

## For AI Assistants (Claude, GPT, etc.)

This guide helps AI agents effectively work with the HybridRAG codebase.

## Quick Context

**What is HybridRAG?**
- Python library for RAG with MongoDB Atlas + Voyage AI
- Combines vector search + keyword search + knowledge graphs
- Single database architecture (MongoDB only)
- Production-ready with comprehensive testing

**Key Facts:**
- This is a **Python library**, not a full-stack web app
- No frontend UI (Streamlit/Next.js) - only CLI and API
- MongoDB Atlas is the single database for everything
- Uses MongoDB 8.2 features: `$rankFusion`, `$vectorSearch`

## Project Navigation

### Where is... ?

| What | Location |
|------|----------|
| Main RAG implementation | `src/hybridrag/core/rag.py` |
| MongoDB hybrid search | `src/hybridrag/enhancements/mongodb_hybrid_search.py` |
| Filter builders | `src/hybridrag/enhancements/filters.py` |
| System prompts | `src/hybridrag/prompts/` |
| CLI commands | `src/hybridrag/cli/app.py` |
| Tests | `tests/` |
| Examples | `examples/` |
| Notebooks | `notebooks/` |

### Common Tasks

**Testing:**
```bash
make test              # All tests
make test-quick        # Fast unit tests
pytest -m p1           # High priority tests
```

**Running Examples:**
```bash
cd examples/
python 01_quickstart.py
```

**CLI:**
```bash
hybridrag --help
hybridrag chat
hybridrag query "What is MongoDB?"
```

## Key Patterns

### MongoDB 8.2 Filter Syntax (CRITICAL)

**Two Different Syntaxes:**

1. **Vector Search** (Standard MongoDB)
```python
# Uses standard MongoDB operators: $eq, $gte, $lte, $in
filters = {
    "metadata.category": {"$eq": "features"},
    "metadata.timestamp": {"$gte": start_date},
}
```

2. **Atlas Search** (Atlas-Specific)
```python
# Uses Atlas Search operators: equals, range
filters = [
    {"equals": {"path": "metadata.category", "value": "features"}},
    {"range": {"path": "metadata.timestamp", "gte": start_date}},
]
```

**Never mix these syntaxes!** See `docs/patterns.md` for examples.

### Search Modes

| Mode | When to Use |
|------|-------------|
| `vector` | Semantic similarity, conceptual queries |
| `keyword` | Exact matching, technical terms |
| `hybrid` | Default - best of both (recommended) |

### Prompts Module

Located in `src/hybridrag/prompts/`:
- `system_prompt.py` - SYSTEM_PROMPT (full), SYSTEM_PROMPT_COMPACT
- `reranking_prompt.py` - Query type detection, reranking instructions
- `entity_extraction_prompt.py` - KG entity extraction
- `memory_prompt.py` - Conversation summarization
- `topic_extraction_prompt.py` - Topic tagging

## Making Changes

### Adding a Feature

1. **Understand**: Read relevant files, check patterns
2. **Plan**: What files need changes? What tests?
3. **Implement**: Follow existing patterns
4. **Test**: Write tests first (TDD) if complex
5. **Verify**: `make test` and `make lint`

### Modifying Core Logic

**Before changing `src/hybridrag/core/rag.py`:**
- This is the main user-facing class
- Changes affect all users
- Ensure backward compatibility
- Add deprecation warnings if removing features

### Adding MongoDB Features

**Add to `src/hybridrag/enhancements/`:**
- Follow existing patterns in `mongodb_hybrid_search.py`
- Use filter builders for consistent syntax
- Add comprehensive tests
- Document in docstrings

## Testing Strategy

### Test Levels

1. **Unit Tests** (`tests/enhancements/`)
   - Fast (< 1s per test)
   - No external dependencies
   - Mock MongoDB/APIs

2. **Integration Tests** (`tests/integration/`)
   - Require MongoDB connection
   - Test full pipelines
   - Mark with `@pytest.mark.integration`

3. **Benchmarks** (`tests/benchmarks/`)
   - Measure performance
   - Mark with `@pytest.mark.benchmark`
   - Compare against baseline

### Test Markers

```python
@pytest.mark.p1           # High priority
@pytest.mark.p2           # Medium priority
@pytest.mark.p3           # Low priority
@pytest.mark.integration  # Needs MongoDB
@pytest.mark.benchmark    # Performance test
```

## Common Pitfalls

### 1. Filter Syntax Confusion
```python
# ❌ WRONG: Atlas syntax in vector search
{"equals": {"path": "category", "value": "features"}}

# ✅ RIGHT: MongoDB syntax in vector search
{"category": {"$eq": "features"}}
```

### 2. Missing Async/Await
```python
# ❌ WRONG: Forgot await
results = rag.query(query="test")

# ✅ RIGHT: Async context
results = await rag.query(query="test")
```

### 3. Hardcoded Values
```python
# ❌ WRONG: Hardcoded connection
MONGODB_URI = "mongodb://localhost:27017"

# ✅ RIGHT: From environment
settings = get_settings()
client = MongoClient(settings.MONGODB_URI)
```

### 4. Naive Datetime
```python
# ❌ WRONG: Deprecated, no timezone
timestamp = datetime.utcnow()

# ✅ RIGHT: Timezone-aware
timestamp = datetime.now(timezone.utc)
```

## Code Style

**Follow existing patterns:**
- Black formatting (88 char line length)
- Isort for imports
- Type hints for all public functions
- Google-style docstrings

**Example:**
```python
async def query(
    self,
    query: str,
    mode: str = "hybrid",
    top_k: int = 10,
) -> list[SearchResult]:
    """
    Execute a search query.

    Args:
        query: User query text
        mode: Search mode (vector, keyword, hybrid)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [romiluz13/Hybrid-Search-RAG](https://github.com/romiluz13/Hybrid-Search-RAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
