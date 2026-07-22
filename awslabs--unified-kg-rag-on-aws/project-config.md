---
trigger: always_on
description: AWS-native Knowledge Graph RAG framework. Reimplements the Microsoft GraphRAG
---

# Project Guide for Claude Code

AWS-native Knowledge Graph RAG framework. Reimplements the Microsoft GraphRAG
and LightRAG methodologies on Bedrock + Neptune + OpenSearch, with DynamoDB for
incremental-indexing state.

## Architecture (hexagonal / ports & adapters)

The package is laid out in explicit hexagonal layers so the two RAG
methodologies (GraphRAG community-summary, LightRAG dual-level keyword) share
one ingestion/indexing/caching/hybrid-search infrastructure and only differ at
the algorithm layer. Dependencies point **inward**: `application → adapters →
ports → domain`, with `shared` as a cross-cutting kernel any layer may use.

- **`domain/`**: technology-agnostic core — `models/` (Pydantic), pure
  algorithms (`ingestion/` merge/delta/resolve/analyze, `retrieval/`
  strategy registry + mixins), and `prompts/` templates. No boto3/LangChain/
  backend imports (enforced by review; verifiable with grep).
- **`ports/`**: the abstract interfaces the domain depends on — `DocStatusPort`
  (`Protocol`) and the write-side indexer ABCs `BaseIndexer`/`GraphIndexer`/
  `VectorIndexer` (+ `IndexingStats`). `ports/__init__` is the port catalog and
  documents that the retrieval/evaluation abstract bases are *adapter bases*
  (they construct infra in `__init__`) re-exported for discovery.
- **`adapters/`**: concrete technology bindings — `aws/` (Bedrock, Neptune,
  OpenSearch, DynamoDB, S3), `storage/` (indexers), `retrievers/`,
  `search_strategies/`, `retrieval/` (base + hybrid scorer + token/memory
  managers), `ingestion/` (LLM/IO-coupled chunker/extractor/loader/parser/
  translator), `renderers/`, `evaluators/`.
- **`application/`**: orchestration + entry points — `cli/` (pyproject scripts
  resolve to `unified_kg_rag.application.cli.*`), `ingestion/` (pipeline + stages),
  `storage/indexing_manager`, `retrieval/rag_chain`.
- **`shared/`**: cross-cutting kernel — config, logging, exceptions, metrics,
  cache/pipeline managers, `utils/`.
- **`evaluation/` and `visualization/`**: real logic packages (evaluators +
  manager; render loop + `embeddings/`/`exporters/`/`renderers/`). Import
  everything else from its real location (`application.retrieval.rag_chain`,
  `application.storage.indexing_manager`, `application.ingestion.pipeline`,
  `adapters.*`, `domain.*`).
- **Registries over hardcoded dispatch**: search strategies register via
  `@register_strategy` (`domain/retrieval/strategy_registry.py`). Follow this
  pattern — and `ParserFactory.register_loader` /
  `EvaluationManager._resolve_evaluator_class` — instead of `if/elif` dispatch.

See `docs/design.md` §2 for the full layer map and dependency rule.

### Adding things
- **New search strategy**: subclass `BaseSearchStrategy`, decorate with
  `@register_strategy(SearchStrategy.X, required_roles=(...))`, export from
  `adapters/search_strategies/__init__.py`. No edits to `rag_chain` needed.
- **New storage/LLM backend**: implement the relevant port and inject it —
  `IndexingManager(vector_indexer=…, graph_indexer=…)`,
  `GraphRAGChain(model_factory=…, retriever_builders=…)`,
  `*(embedding_factory=…)`. Defaults stay Bedrock/Neptune/OpenSearch; don't
  hardcode a backend into a manager's `__init__`. (See design.md §15 "Custom
  backends".)
- **New parser / file format**: `ParserFactory.register_loader(".ext", Loader)`
  with any LangChain `BaseLoader` subclass — auto-discovered and parseable, no
  factory edit.
- **New evaluator**: subclass `BaseGraphRAGEvaluator`, add a branch in
  `EvaluationManager._resolve_evaluator_class`, add an `EvaluatorType` enum.
- **New config section**: add a Pydantic `BaseModel`, attach it to its parent
  via `Field(default_factory=...)`, document it in `config-template.yaml`.

## Capabilities & CLIs

- **Two retrieval methodologies** (user-selectable via `RAGInput.search_strategy`):
  GraphRAG community-summary (`auto`/`drift`/`global`/`local`/`simple`) and
  LightRAG dual-level keyword (`mix`/`hybrid`/`naive`). Both share the same
  ingestion, indexing, caching, multilingual, and hybrid-scoring infrastructure;
  only the retrieval algorithm differs. LightRAG `mix`/`hybrid` extract high/low
  keywords (`KeywordsExtractionPrompt`) and query a relationship vector index
  (high-level) + entity index (low-level) + Neptune expansion.
- **Incremental indexing**: enable `aws.dynamodb` to diff a corpus by content
  hash and only (re)index new/changed documents, merging into the live graph
  (`IncrementalIndexer`, `ingestion/merge/`, idempotent `upsert_*`/`delete_by_id`
  on both indexers). Deletions remove a document's *exclusive* artifacts via
  per-document lineage in the DynamoDB registry.
- **CLIs** (`pyproject` scripts): `run-ingestion`, `run-rag`, `run-eval`,
  `run-visualization` (render from exported graph data, no ingestion),
  `run-prompt-tuning` (profile a corpus → domain-adapted `custom_prompts` YAML).
- **Evaluation**: `langchain` + `ragas` (text similarity) plus `graph_aware`
  (entity/relationship coverage = recall, from ground-truth
  `expected_entities`/`expected_relationships`; precision/F1 intentionally not
  emitted — see the evaluator docstring). Add an evaluator by subclassing
  `BaseGraphRAGEvaluator` + a branch in `EvaluationManager._resolve_evaluator_class`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awslabs/unified-kg-rag-on-aws](https://github.com/awslabs/unified-kg-rag-on-aws) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
