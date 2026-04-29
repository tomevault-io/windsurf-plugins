---
trigger: always_on
description: MinerU Document Explorer 是 MinerU 团队基于 QMD 进行的二次开发，定位为
---

# MinerU Document Explorer — Agent-Native Knowledge Engine

MinerU Document Explorer 是 MinerU 团队基于 QMD 进行的二次开发，定位为
**Agent 原生的知识库系统**。融合 Karpathy LLM Wiki 知识编译思想和 MinerU
高精度文档理解能力，为 AI Agent 提供三组工具：信息检索、文档精读、知识摄取，
构成索引 → 检索 → 精读的完整闭环。

Use Bun instead of Node.js (`bun` not `node`, `bun install` not `npm install`).

## Commands

```sh
qmd collection add . --name <n> # Create/index collection
qmd collection list # List all collections with details
qmd collection remove <name> # Remove a collection by name
qmd collection rename <old> <new> # Rename a collection
qmd ls [collection[/path]] # List collections or files in a collection
qmd context add [path] "text" # Add context for path (defaults to current dir)
qmd context list # List all contexts
qmd cleanup # Clean up caches, vacuum DB
qmd context rm <path> # Remove context
qmd get <file> # Get document by path or docid (#abc123)
qmd multi-get <pattern> # Get multiple docs by glob or comma-separated list
qmd status # Show index status and collections
qmd update [--pull] # Re-index all collections (--pull: git pull first)
qmd embed # Generate vector embeddings (uses node-llama-cpp)
qmd query <query> # Search with query expansion + reranking (recommended)
qmd search <query> # Full-text keyword search (BM25, no LLM)
qmd vsearch <query> # Vector similarity search (no reranking)
qmd mcp # Start MCP server (stdio transport)
qmd mcp --http [--port N] # Start MCP server (HTTP, default port 8181)
qmd mcp --http --daemon # Start as background daemon
qmd mcp stop # Stop background MCP daemon
qmd doc-toc <file> # Document table of contents
qmd doc-read <file> <addr..> # Read content at specific addresses
qmd doc-grep <file> <pattern> # Regex/keyword search within a document
qmd wiki init <collection> # Mark collection as wiki type (LLM-maintained)
qmd wiki ingest <source> # Analyze source doc for wiki page creation
qmd wiki write <coll> <path> # Write a wiki page from stdin
qmd wiki lint # Health-check: orphans, broken links, stale pages
qmd wiki log [since-date] # View wiki activity timeline
qmd wiki index <collection> # Generate wiki index page
```

## Collection Management

```sh
# List all collections
qmd collection list

# Create a collection with explicit name
qmd collection add ~/Documents/notes --name mynotes --mask '**/*.md'

# Index PDF/DOCX/PPTX files (requires Python 3 + pymupdf/python-docx/python-pptx)
qmd collection add ~/papers --name papers --mask '**/*.{md,pdf,docx,pptx}'

# Remove a collection
qmd collection remove mynotes

# Rename a collection
qmd collection rename mynotes my-notes

# List all files in a collection
qmd ls mynotes

# List files with a path prefix
qmd ls journals/2025
qmd ls qmd://journals/2025
```

## Context Management

```sh
# Add context to current directory (auto-detects collection)
qmd context add "Description of these files"

# Add context to a specific path
qmd context add /subfolder "Description for subfolder"

# Add global context to all collections (system message)
qmd context add / "Always include this context"

# Add context using virtual paths
qmd context add qmd://journals/ "Context for entire journals collection"
qmd context add qmd://journals/2024 "Journal entries from 2024"

# List all contexts
qmd context list

# Check for collections or paths without context
qmd context check

# Remove context
qmd context rm qmd://journals/2024
qmd context rm /  # Remove global context
```

## Wiki (LLM Wiki Pattern)

QMD supports the LLM Wiki pattern — a persistent, compounding knowledge base
where an LLM incrementally builds and maintains interlinked wiki pages from raw
source documents.

Collections have a `type` field: `raw` (default, immutable sources) or `wiki`
(LLM-generated pages). MCP tools `wiki_ingest`, `wiki_lint`, `wiki_log`, and
`wiki_index` provide the full wiki lifecycle.

```sh
# Create a wiki collection
qmd collection add ~/wiki --name mywiki --type wiki

# Or mark an existing collection as wiki
qmd wiki init mywiki

# Analyze a source document for wiki processing
qmd wiki ingest sources/paper.pdf --wiki mywiki

# Write a wiki page from stdin
echo '# Summary of paper' | qmd wiki write mywiki sources/paper.md
cat page.md | qmd wiki write mywiki concepts/topic.md --title "Topic Name"

# Health-check the wiki (orphans, broken links, missing pages)
qmd wiki lint

# View wiki activity log
qmd wiki log
qmd wiki log 2025-01-01

# Generate wiki index page
qmd wiki index mywiki
```

### MCP Tools

Core tools:
- `query` — Search with plain query or typed sub-queries (lex/vec/hyde)
- `get` — Retrieve document by path or docid
- `multi_get` — Batch retrieve by glob, comma-separated list, or docids
- `status` — Index health and collection info

Document tools (MinerU Document Explorer):
- `doc_toc` — Document table of contents (headings/bookmarks/slides)
- `doc_read` — Read content at specific addresses from doc_toc/doc_grep/doc_query
- `doc_grep` — Regex/keyword search within a single document
- `doc_query` — Semantic search within a single document
- `doc_elements` — Extract tables, figures, equations

Writing tools:
- `doc_write` — Write documents (auto-logged for wiki collections)
- `doc_links` — Forward/backward link graph for a document

Wiki tools:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opendatalab/MinerU-Document-Explorer](https://github.com/opendatalab/MinerU-Document-Explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-19 -->
