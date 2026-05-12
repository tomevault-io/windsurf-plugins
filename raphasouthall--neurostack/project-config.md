---
trigger: always_on
description: NeuroStack is a neuroscience-grounded knowledge management system. CLI + MCP server + OpenAI-compatible API. Everything runs locally against a Markdown vault indexed in SQLite + FTS5.
---

# NeuroStack - Claude Code Guide

NeuroStack is a neuroscience-grounded knowledge management system. CLI + MCP server + OpenAI-compatible API. Everything runs locally against a Markdown vault indexed in SQLite + FTS5.

## Quick Reference

### Installation

```bash
npm install -g neurostack    # bootstraps CLI, Python, uv, deps
neurostack init              # cloud/local → lite/full → vault setup → index
```

### MCP Server (recommended for Claude Code)

Add to `~/.claude/.mcp.json`:
```json
{
  "mcpServers": {
    "neurostack": {
      "command": "neurostack",
      "args": ["serve"],
      "env": {}
    }
  }
}
```

### OpenAI-compatible API

```bash
pip install neurostack[api]
neurostack api                          # localhost:8000
neurostack api --host 0.0.0.0 --port 9000
NEUROSTACK_API_KEY=secret neurostack api  # with auth
```

Endpoints: `POST /v1/chat/completions`, `POST /v1/embeddings`, `GET /v1/models`, `GET /health`
Models: `neurostack-ask` (RAG), `neurostack-search` (hybrid), `neurostack-tiered` (auto-depth), `neurostack-triples` (facts)

## CLI Commands

### Search & Retrieval
| Command | Description |
|---------|-------------|
| `neurostack search "query"` | Hybrid FTS5 + semantic search. Flags: `--mode hybrid\|semantic\|keyword`, `--top-k N`, `--context "domain"`, `--workspace "path/"` |
| `neurostack ask "question"` | RAG Q&A with inline `[[citations]]`. Uses Ollama LLM. Flags: `--top-k N`, `--workspace` |
| `neurostack tiered "query"` | Token-efficient tiered retrieval. `--depth triples\|summaries\|full\|auto` |
| `neurostack triples "query"` | Search knowledge graph triples (subject-predicate-object facts) |
| `neurostack summary "note.md"` | Get pre-computed AI summary of a note |
| `neurostack graph "note.md"` | Wiki-link neighborhood with PageRank scores. `--depth N` |
| `neurostack related "note.md"` | Find semantically similar notes by embedding distance |
| `neurostack communities query "question"` | GraphRAG global queries across topic clusters |
| `neurostack context "task description"` | Assemble task-scoped context for session recovery |
| `neurostack brief` | Compact session briefing (recent activity, hot notes, alerts) |

### Memory Management
| Command | Description |
|---------|-------------|
| `neurostack memories add "content"` | Save a memory. `--tags "a,b"`, `--type observation\|decision\|convention\|learning\|context\|bug`, `--workspace`, `--ttl N` (hours) |
| `neurostack memories search "query"` | Search memories. `--type`, `--workspace`, `--limit N` |
| `neurostack memories list` | List recent memories |
| `neurostack memories update ID` | Update memory. `--content`, `--tags`, `--add-tags`, `--remove-tags`, `--type` |
| `neurostack memories forget ID` | Delete a memory |
| `neurostack memories merge TARGET SOURCE` | Merge two memories (unions tags, audit trail) |
| `neurostack memories prune` | Clean up. `--expired` or `--older-than N` (days) |
| `neurostack memories stats` | Memory statistics |
| `neurostack capture "thought"` | Quick-capture to vault inbox. `--tags "a,b"` |

### Sessions
| Command | Description |
|---------|-------------|
| `neurostack sessions start` | Begin a memory session. `--source "agent-name"`, `--workspace` |
| `neurostack sessions end ID` | End session. `--summarize` for LLM summary |
| `neurostack sessions list` | List recent sessions. `--workspace`, `--limit N` |
| `neurostack sessions show ID` | Session details and memories |
| `neurostack sessions search "query"` | Search session transcripts (delegates to session-index) |

### Indexing & Maintenance
| Command | Description |
|---------|-------------|
| `neurostack index` | Full re-index. `--skip-summary`, `--skip-triples` |
| `neurostack watch` | File watcher - auto-indexes on vault changes |
| `neurostack reembed-chunks` | Re-embed all chunks with contextual text |
| `neurostack backfill` | Backfill missing summaries and/or triples |
| `neurostack folder-summaries` | Build folder-level summaries for context boosting |
| `neurostack communities build` | Run attractor basin community detection |
| `neurostack harvest` | Extract insights from recent Claude Code sessions |
| `neurostack record-usage "path1" "path2"` | Record note usage for hotness scoring |
| `neurostack decay` | Report note excitability and dormancy |
| `neurostack prediction-errors` | Show notes flagged as poor retrieval fit |

### Setup & Diagnostics
| Command | Description |
|---------|-------------|
| `neurostack init [path]` | One-command setup: cloud/local, lite/full, deps, vault, index. `--mode lite\|full`, `--cloud`, `--profession`, `--pull-models` |
| `neurostack scaffold [profession]` | Apply profession pack to existing vault. `--list` to see options |
| `neurostack onboard /path/to/notes` | Onboard existing Markdown notes. `--dry-run`, `--no-index` |
| `neurostack install` | **(Deprecated)** Use `neurostack init` instead |
| `neurostack update` | Pull latest source and re-sync deps |
| `neurostack doctor` | Validate all subsystems. `--strict` exits 1 on missing vault/db |
| `neurostack status` | Show current status |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [raphasouthall/neurostack](https://github.com/raphasouthall/neurostack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
