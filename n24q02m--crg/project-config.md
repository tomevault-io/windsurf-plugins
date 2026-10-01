---
trigger: always_on
description: Fork of code-review-graph with fixed multi-word search, qualified call resolution,
---

# better-code-review-graph

Fork of code-review-graph with fixed multi-word search, qualified call resolution,
dual-mode embedding (ONNX local + cloud chain via `EMBEDDING_MODELS`), and output pagination.
See `AGENTS.md` va `README.md` de hieu architecture va configuration.

## Cau truc

- `src/better_code_review_graph/` -- Package chinh (src layout)
  - `server.py` -- FastMCP server, 6 tools: graph + query + review (3 main) + config + security + help
  - `tools.py` -- MCP tool implementations (build, query, impact, review, search, embed, stats, docs, large functions)
  - `parser.py` -- Tree-sitter parsing (14 langs) + call target resolution
  - `graph.py` -- SQLite GraphStore, search, impact radius, NetworkX cache
  - `incremental.py` -- Git integration, file watching, incremental updates
  - `embeddings.py` -- Dual-mode embedding: local ONNX through the fastretrieval registry + cloud chain (`EMBEDDING_MODELS`) via OpenAI-compatible HTTP clients (`hull_core.providers`)
  - `docs/` -- Help tool documentation (graph.md, query.md, review.md, config.md)
- `cli.py` -- local CLI: no args starts MCP stdio; positional subcommands expose graph/query/review/security over the same domain services

  - `__init__.py` -- Version export
  - `__main__.py` -- `python -m` entry (calls cli.main)
  - `py.typed` -- PEP 561 marker
- `tests/` -- Mirror source modules
- `skills/` -- Claude Code skills (impact-audit, onboard-repo, refactor-check, review-delta, review-pr, security-sweep)
- `hooks/` -- SessionStart + UserPromptSubmit + PostToolUse hooks
- `.claude-plugin/` -- Plugin manifest + marketplace metadata

## Local-first boundary

- CLI and bundled Skills are the primary coding-harness surfaces.
- MCP stdio is a secondary protocol adapter; it must not duplicate graph logic.
- Graph state is local at `<repo>/.code-review-graph/graph.db` unless explicit
  self-host/multi-user configuration changes the data directory.
- CRG has no hosted Cloudflare runtime in the target topology. PyPI, CI,
  security, GitHub releases, and eligible stable MCP Registry publication remain
  active. Historical OCI tags remain available; new public OCI images do not.

## Lenh thuong dung

```bash
uv sync --group dev                # Cai dependencies
uv run pytest                      # Test tat ca
uv run pytest tests/test_graph.py::test_function_name -v  # Test don le
uv run ruff check .                # Lint
uv run ruff format .               # Format
uv run ruff check --fix . && uv run ruff format .  # Fix
uv run ty check                    # Type check (ty lenient config)
uv run better-code-review-graph        # Chay MCP server (stdio, default)
```

## Cau hinh quan trong

- **Python 3.13 bat buoc** -- `requires-python = "==3.13.*"`
- Ruff: line-length 88, target py313, rules E/F/W/I/UP/B/C4, ignore E501
- ty: lenient (unresolved-import, unresolved-attribute, possibly-missing-attribute all "ignore")

## Architecture

```
Source files --> Tree-sitter parser --> SQLite graph (nodes + edges)
                                          |
                                     NetworkX BFS --> Impact radius
                                          |
                                     Embedding store --> Semantic search
                                          |
                                     FastMCP server --> secondary MCP adapter (6 tools: graph + query + review + config + security + help)
```

- **Parser** (parser.py): Tree-sitter extracts nodes (File, Class, Function, Type, Test) and edges (CALLS, IMPORTS_FROM, INHERITS, IMPLEMENTS, CONTAINS, TESTED_BY, DEPENDS_ON). Resolves same-file bare call targets to qualified names.
- **Graph** (graph.py): SQLite with WAL mode. Multi-word AND-logic search. GraphNode/GraphEdge dataclasses.
- **Incremental** (incremental.py): Git diff detection, file hash tracking, re-parses only changed files.
- **Embeddings** (embeddings.py): Dual-mode -- local ONNX through the fastretrieval registry (default, zero-config) or cloud via the `EMBEDDING_MODELS` chain (OpenAI-compatible HTTP clients; order = fallback, empty = local). Fixed 768-dim storage.
- **Tools** (tools.py): Implementation layer for all graph operations. Output pagination via max_results.
- **Server** (server.py): 6 tools — graph (build/update/stats/embed/export/summarize), query (query/search/impact/large_functions/spot_check/renamed_in_diff/diff), review, config, security, help. Returns structured dict payloads over MCP; the CLI serializes them as JSON.

## Embedding backends

Embedding (cloud backend) + the LLM summarizer dispatch through
OpenAI-compatible HTTP clients (`hull_core.providers.openai_spec`).

Per-task model chains, CSV `provider/model,provider/model`, order = fallback. Provider is inferred from the model prefix.

- `EMBEDDING_MODELS` -- chain embedding. Empty = local ONNX from the fastretrieval built-in registry.
- **Local (default)**: fastretrieval ONNX registry -- zero-config, ~570MB download on first use, 768-dim MRL truncation
- API key follows the `<PROVIDER>_API_KEY` convention. The 7 providers documented below:

  | model prefix | key env var | get it at |
  |---|---|---|
  | `gemini/` | `GEMINI_API_KEY` | aistudio.google.com/apikey |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [n24q02m/crg](https://github.com/n24q02m/crg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
