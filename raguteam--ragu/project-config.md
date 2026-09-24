---
trigger: always_on
description: - [Communication Language](#communication-language)
---

# AGENTS.md — RAGU Development Guide

## Table of Contents

- [Communication Language](#communication-language)
- [Project Overview](#project-overview)
- [Development Commands](#development-commands)
- [Workflow After Changes](#workflow-after-changes)
- [Code Conventions](#code-conventions)
- [Testing](#testing)
- [Key Invariants](#key-invariants)
- [Storage Contracts](#storage-contracts)
- [Prompt System](#prompt-system)
- [Extending RAGU](#extending-ragu)
- [Do Not Touch](#do-not-touch)
- [Code Quality](#code-quality)

---

## Communication Language

Reply to the user in the language they used (English, Russian, or any other). **Code comments must always be in English** regardless. Log messages and user-facing documentation match the user's language.

---

## Project Overview

RAGU (Retrieval-Augmented Graph Utility) is a modular GraphRAG engine for building, storing, and querying knowledge graphs from text. Entity and relation types follow the [NEREL](https://github.com/nerel-ds/NEREL) schema. Both English and Russian are supported via `Settings.language`.

Processing pipeline:

```
Documents -> Chunker -> List[Chunk]
  -> ArtifactExtractor -> List[Entity], List[Relation]
    -> EntitySummarizer / RelationSummarizer (merge + optional LLM summarization)
      -> Optional GraphBuilderModules (e.g., RemoveIsolatedNodes)
        -> Leiden community detection -> List[Community]
          -> CommunitySummarizer -> List[CommunitySummary]
            -> Index (persists graph + KV + vectors)

Query -> SearchEngine.a_search() -> retrieval context
      -> SearchEngine.a_query() -> LLM-generated answer
```

The full public API is re-exported from `ragu/__init__.py`. Top-level subpackages: `chunker`, `common`, `graph`, `models`, `search_engine`, `storage`, `triplet`, `utils`.

---

## Development Commands

The project uses **`uv`** for package management.

```bash
# Editable install
uv pip install -e .

# With test dependencies
uv pip install -e ".[test]"

# Full test suite (with coverage, per pytest.ini)
pytest tests/

# Fast tests only
pytest -m "not slow and not integration"

# Without coverage
pytest -q --no-cov
```

---

## Workflow After Changes

After modifying code, the agent **must**:

1. Run `pytest -m "not slow and not integration"` and ensure it passes.
2. If a new public class/function was added — re-export it from the subpackage `__init__.py` **and** from `ragu/__init__.py`.
3. If a new prompt was added — register it in `DEFAULT_PROMPT_TEMPLATES` (`ragu/common/prompts/prompt_storage.py`). There is no auto-discovery.
4. If a new dependency was added — update `pyproject.toml`.
5. **Documentation — accuracy.** If you change a public API (rename/remove a class, function, parameter, default, or prompt name), update every doc example that references it. Grep the old symbol across `*.md` (exclude `.venv`, `.git`, `node_modules`) and fix the matches. Examples must stay runnable.
6. **Documentation — coverage.** If you add a new public class/function, storage adapter, search engine, builder module, extractor, or `Settings` field — document it at its **canonical home** (see table below) and link from the relevant index. Do **not** duplicate the example in several files: one source of truth, linked elsewhere. Re-export per rule 2 still applies.
7. **Documentation — bilingual parity.** Conceptual docs in `docs/{en,ru}/` must be updated in **both** languages **atomically, within the same change**. Subpackage `ragu/*/README.md` are English-only and are not mirrored.

> **Carve-out.** Purely internal changes — refactors, performance work, bug fixes with no change to public API, defaults, behavior, or the NEREL ontology — do **not** require documentation updates.

**Canonical documentation homes** (update the home, link everywhere else):

| Change | Canonical home |
|---|---|
| NEREL entity/relation types | `docs/{en,ru}/ontology.md` (+ `ragu/triplet/types.py`) |
| RAGU-lm prompts/examples | `docs/{en,ru}/ragu_lm.md` |
| Conceptual workflow, search strategies | `docs/{en,ru}/ragu_components.md` |
| New prompt (name + schema) | `DEFAULT_PROMPT_TEMPLATES` (rule 3) + table in `ragu/common/prompts/README.md` |
| `Settings` field / token limit | `ragu/common/README.md` (+ serialization list if user-configurable) |
| `CachedAsyncOpenAI` / `LLMOpenAI` / `EmbedderOpenAI` parameter | `ragu/models/README.md` |
| Chunker / builder / extractor / search engine | matching `ragu/<sub>/README.md` + re-export |
| Storage adapter | `ragu/storage/<sub>/README.md` |
| Public symbol | `ragu/__init__.py` (+ subpackage `__init__.py`) |

After edits, verify in text: (a) `grep` of any removed/renamed symbol across `*.md` returns nothing, and (b) no local markdown links are broken.

---

## Code Conventions

### Imports
- **Absolute imports only**: `from ragu.common.logger import logger`. Relative imports (`from ..common import ...`) are forbidden.

### Naming
- **Classes**: PascalCase.
- **Functions/methods**: snake_case.
- **Async methods**: prefixed with `a_` (`a_search`, `a_embed_text`).
- **Sync wrappers** of async methods: no prefix, delegate via `always_get_an_event_loop()`.
- **Constants**: UPPER_SNAKE_CASE.
- **Private/internal**: single underscore prefix.
- **Files**: snake_case.

### Dataclasses

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RaguTeam/RAGU](https://github.com/RaguTeam/RAGU) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
