---
trigger: always_on
description: Local knowledge graph + intelligence layer for Obsidian vaults. Rust CLI + MCP server. llama.cpp inference with Metal GPU. MIT licensed.
---

# engraph

Local knowledge graph + intelligence layer for Obsidian vaults. Rust CLI + MCP server. llama.cpp inference with Metal GPU. MIT licensed.

## Architecture

Single binary with 26 modules behind a lib crate:

- `config.rs` — loads `~/.engraph/config.toml` and `vault.toml`, merges CLI args, provides `data_dir()`. Includes `intelligence: Option<bool>`, `[models]` section for model overrides, `[obsidian]` section (CLI path, enabled flag), and `[agents]` section (registered AI agent names). `Config::save()` writes back to disk.
- `chunker.rs` — smart chunking with break-point scoring algorithm. Finds optimal split points considering headings, code fences, blank lines, and thematic breaks. `split_oversized_chunks()` handles token-aware secondary splitting with overlap
- `docid.rs` — deterministic 6-char hex IDs for files (SHA-256 of path, truncated). Shown in search results for quick reference
- `llm.rs` — ML inference via llama.cpp (Rust bindings: `llama-cpp-2`). Three traits: `EmbedModel` (embeddings), `RerankModel` (cross-encoder scoring), `OrchestratorModel` (query intent + expansion). Three llama.cpp implementations: `LlamaEmbed` (embeddinggemma-300M GGUF on Metal GPU), `LlamaOrchestrator` (Qwen3-0.6B for query analysis + expansion), `LlamaRerank` (Qwen3-Reranker-0.6B for relevance scoring). Global `LlamaBackend` via `OnceLock`. Also: `MockLlm` for testing, `HfModelUri` for model download, `FlexTokenizer` (HuggingFace tokenizers + shimmytok GGUF fallback), `PromptFormat` for model-family prompt templates, `heuristic_orchestrate()` fast path, `LaneWeights` per query intent
- `fts.rs` — FTS5 full-text search support. Re-exports `FtsResult` from store. BM25-ranked keyword search
- `fusion.rs` — Reciprocal Rank Fusion (RRF) engine. Merges semantic + FTS5 + graph + reranker results. Supports per-lane weighting, `--explain` output with intent + per-lane detail
- `markdown.rs` — section parser. Heading detection (ATX `#` headings with level tracking), section extraction by heading text, frontmatter splitting (YAML block between `---` fences). Powers section-level reading and editing
- `migrate.rs` — PARA migration engine. Heuristic classification of vault notes into Projects/Areas/Resources/Archive using priority-ordered rules (tasks, active status, recurring topics, people, reference, done/inactive). Preview-then-apply workflow generates markdown + JSON preview for review before moving files. Rollback support via `engraph migrate para --undo` reverses the last migration using SQLite migration log. Three MCP tools (`migrate_preview`, `migrate_apply`, `migrate_undo`) and three HTTP endpoints (`POST /api/migrate/preview`, `/apply`, `/undo`)
- `obsidian.rs` — Obsidian CLI wrapper. Process detection (checks if Obsidian is running), circuit breaker state machine (Closed/Degraded/Open) for resilient CLI delegation, async subprocess execution with timeout. Falls back gracefully when Obsidian is unavailable
- `health.rs` — vault health diagnostics. Orphan detection (notes with no incoming or outgoing wikilinks), broken link detection (wikilinks pointing to nonexistent notes), stale note detection (notes not modified within configurable threshold), tag hygiene (unused/rare tags). Returns structured health report
- `context.rs` — context engine. Seven functions: `read` (full note content + metadata), `read_section` (targeted section extraction by heading), `list` (filtered note listing with `created_by` filter), `vault_map` (structure overview), `who` (person context bundle), `project` (project context bundle), `context_topic` (rich topic context with budget trimming). Pure functions taking `ContextParams` — no model loading except `context_topic` which reuses `search_internal`
- `vecstore.rs` — sqlite-vec virtual table integration. Manages the `vec_chunks` vec0 table for vector storage and KNN search. Handles insert, delete, and search operations against the virtual table
- `tags.rs` — tag registry module. Maintains a `tag_registry` table tracking known tags with source attribution. Supports fuzzy matching for tag suggestions during note creation
- `links.rs` — link discovery module. Three match types: exact basename, fuzzy (sliding window Levenshtein, 0.92 threshold), and first-name (People folder, suggestion-only at 650bp). Overlap resolution via type priority (exact > alias > fuzzy > first-name)
- `placement.rs` — folder placement engine. Uses folder centroids (online mean of embeddings per folder) to suggest the best folder for new notes. Falls back to inbox when confidence is low. Includes placement correction detection (`detect_correction_from_frontmatter`) and frontmatter stripping for moved files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devwhodevs/engraph](https://github.com/devwhodevs/engraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
