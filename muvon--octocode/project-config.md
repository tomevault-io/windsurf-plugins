---
trigger: always_on
description: Rust CLI tool (v0.14.x) that indexes codebases into LanceDB vector stores, builds knowledge graphs via GraphRAG, and exposes everything through an MCP server for AI assistant integration. Core stack: Rust + tokio + LanceDB + tree-sitter + octolib (embedding/LLM providers). Embedding providers and LLM are delegated entirely to `octolib`.
---

# Octocode — AI-Powered Code Intelligence

Rust CLI tool (v0.14.x) that indexes codebases into LanceDB vector stores, builds knowledge graphs via GraphRAG, and exposes everything through an MCP server for AI assistant integration. Core stack: Rust + tokio + LanceDB + tree-sitter + octolib (embedding/LLM providers). Embedding providers and LLM are delegated entirely to `octolib`.

## Project Structure

```
src/
  main.rs                    — CLI entry point, command dispatch
  config.rs                  — All config structs; NO runtime defaults (panic on missing)
  storage.rs                 — Project DB path resolution (~/.local/share/octocode/<hash>/)
  store/                     — LanceDB operations
    mod.rs                   — Store struct, block types (CodeBlock, TextBlock, DocumentBlock, CommitBlock)
    vector_optimizer.rs      — Auto-tuned IVF_RQ / IVF_HNSW_SQ index selection
    batch_converter.rs       — Arrow RecordBatch construction
    table_ops.rs             — content_exists(), low-level table ops
    graphrag.rs              — GraphRAG node storage
    metadata.rs              — Table metadata helpers
  embedding/mod.rs           — Thin wrapper over octolib; generate_embeddings*, hash utilities
  llm/mod.rs                 — Thin wrapper over octolib LLM; LlmClient::from_config()
  indexer/
    mod.rs                   — NoindexWalker, file pipeline orchestration
    languages/               — One file per language, all implement Language trait
    graphrag/                — GraphRAG builder, AI relationship discovery
    batch_processor.rs       — Batch embedding + flush logic
    differential_processor.rs — Incremental file change processing
    contextual.rs            — LLM-based contextual chunk enrichment
    commits/                 — Git commit history indexing
  mcp/
    server.rs                — McpServer (rmcp-based, stdin + HTTP modes)
    semantic_code.rs         — SemanticCodeProvider tool impl
    graphrag.rs              — GraphRagProvider tool impl
    lsp/                     — LSP integration tools
    proxy.rs                 — Multi-repo MCP proxy
  commands/                  — One file per CLI subcommand
config-templates/default.toml — Single source of truth for ALL config defaults
```

## Where to Look

| Task | Start here |
|------|------------|
| Add/change config option | `config-templates/default.toml` first, then matching struct in `src/config.rs` |
| Add embedding provider/model | `octolib` repo — providers live there, not here |
| Add language support | `src/indexer/languages/{lang}.rs` + register in `languages/mod.rs` |
| Add MCP tool | `src/mcp/semantic_code.rs` or `src/mcp/graphrag.rs` (new provider file if distinct domain) |
| Add CLI command | `src/commands/{cmd}.rs` + register in `commands/mod.rs` + dispatch in `main.rs` |
| Store/query LanceDB | `src/store/mod.rs` (block types, Store methods) + `src/store/table_ops.rs` |
| Vector index tuning | `src/store/vector_optimizer.rs` — automatic, rarely needs changes |
| File discovery / ignore | `src/indexer/mod.rs` → `NoindexWalker` |
| Project DB path logic | `src/storage.rs` |
| Batch embedding pipeline | `src/indexer/batch_processor.rs` |
| GraphRAG relationships | `src/indexer/graphrag/` |
| Git commit indexing | `src/indexer/commits/` |

## How Things Work

### Configuration — No Defaults Rule
Config structs use `panic!()` in `Default::default()` for all non-trivial sections (GraphRAG, Reranker, HybridSearch). This is intentional — every value must come from `config-templates/default.toml`. When adding any config field:
1. Add to struct in `src/config.rs`
2. Add default value in `config-templates/default.toml`
3. Never add a real fallback value in `Default::default()`

### Embedding — Delegated to octolib
All provider implementations live in `octolib`. `src/embedding/mod.rs` is a thin wrapper that:
- Re-exports `octolib::embedding` types
- Adds `generate_embeddings()` / `generate_embeddings_batch()` with 3-attempt exponential backoff
- Adds `generate_search_embeddings()` for mode-aware query embedding (`code` / `docs` / `text` / `commits` / `all`)
- Provides `calculate_content_hash_with_lines()` for block dedup

Model format everywhere: `"provider:model"` (e.g., `"voyage:voyage-code-3"`, `"openrouter:openai/gpt-4o-mini"`).

### LLM — Delegated to octolib
`src/llm/mod.rs` wraps `octolib::llm`. Use `LlmClient::from_config(config)` for the default model, `LlmClient::with_model(config, model_str)` to override. Never call octolib LLM directly from commands — always go through `LlmClient`.

### Store — Block Types and Tables
Four LanceDB tables: `code_blocks`, `text_blocks`, `document_blocks`, `commit_blocks`. Each maps to a typed struct in `src/store/mod.rs`. Schema is created on first use; dimension mismatch triggers automatic table drop + recreate. Table handles are cached in `Arc<RwLock<HashMap>>` — never open tables manually.

```rust
// ✅ Use typed store methods
store.store_code_blocks(&blocks, &embeddings).await?;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Muvon/octocode](https://github.com/Muvon/octocode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
