---
trigger: always_on
description: Cognitive Codebase Matrix (CCM) MCP skill for deep codebase intelligence. Gives AI agents a queryable knowledge graph over any project: semantic hybrid search, call chain tracing, blast-radius analysis, and cursor-aware context retrieval — all via 9 MCP tools. WHEN: understand a large codebase, find callers/callees, map blast radius, retrieve cursor context, trace a call chain, inspect a graph node, get recently changed code, or index a project before starting work.
---


# Cognitive Codebase Matrix (CCM)

> Transform any codebase into a queryable knowledge graph for AI agents.
> CCM combines Tree-sitter AST parsing, LanceDB vector search, and Petgraph graph traversal
> into 9 MCP tools that give agents surgical codebase intelligence.

## When to Activate

Load this skill when you need to:
- Navigate or understand a large, unfamiliar codebase
- Find all callers or callees of a function
- Assess the blast radius before editing a file
- Retrieve code context at a specific file + line position
- Trace the call path between two functions
- Inspect how graph nodes connect to each other
- See what files changed in the last N days (git-aware)
- Index or re-index a project before starting a task

Trigger phrases: "understand the codebase", "find usages of", "what calls X", "who depends on", "blast radius of", "context at line", "trace call chain", "index this project", "recently changed code", "read the graph", "find nodes named".

## When NOT to Use

**DO NOT** activate this skill when:
- The project is **already indexed in memory** from this session and you just need context recall (context is cached; reuse `get_context` directly from memory without re-indexing)
- User is asking for **code refactoring guidance without needing graph traversal** (use code-reviewer skill instead)
- User wants **static type analysis** or **linting feedback** (use language-specific reviewer skill)
- The task is **pure documentation** (no code exploration needed)
- User says "just give me the file" without needing cross-file reasoning

<HARD-GATE>
You MUST call `index_project` before any other tool on a fresh project session.
If the project was already indexed in this conversation, you MAY skip `index_project` and call tools directly.
NEVER assume a project is indexed without explicit mention or evidence.
</HARD-GATE>

## Execution Boundaries

- **Boundary 1:** All file paths MUST be relative to project root (e.g. `src/main.rs`, not `/Users/dev/my-app/src/main.rs`)
- **Boundary 2:** Node IDs MUST be used exactly as returned by tools (never guessed or constructed)
- **Boundary 3:** If a tool returns empty results, check node_id/file format BEFORE escalating
- **Boundary 4:** `max_depth` in trace_call_chain should be 6–15; default (6) may miss distant paths

## MCP Setup

### One-line install (configures Codex, Cursor, Claude Desktop, Antigravity automatically)
```bash
npx @senoldogann/context-manager install
```

### Manual MCP config (VS Code Kilo / any MCP host)
```json
{
  "context-manager": {
    "command": "npx",
    "args": ["-y", "@senoldogann/context-manager", "mcp"],
    "env": { "RUST_LOG": "info" }
  }
}
```

### Minimal environment (~/.ccm/.env)
```ini
# Option A: Local inference (recommended, no API cost)
EMBEDDING_PROVIDER=ollama
EMBEDDING_HOST=http://127.0.0.1:11434
EMBEDDING_MODEL=mxbai-embed-large
EMBEDDING_API_KEY=ollama

# Option B: Cloud (OpenAI)
# EMBEDDING_PROVIDER=openai
# EMBEDDING_API_KEY=sk-your-key
# EMBEDDING_MODEL=text-embedding-3-small

# Security: restrict which projects the MCP server may access
CCM_ALLOWED_ROOTS=/Users/you/projects:/Users/you/sandbox
```

Ollama prerequisites:
```bash
ollama serve
ollama pull mxbai-embed-large
```

## Workflow Modes

CCM supports 4 distinct operational modes depending on your task. Each has a terminal state.

### Mode 1: Semantic Discovery (Entry Point for Exploration)
**When:** You have a vague question like "find auth handling" but don't know the exact file/function.

```
┌─────────────────────────────────────┐
│ index_project (if fresh session)    │
└──────────────┬──────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ search_code (natural language query) │
│ Returns: ranked snippets + node_ids  │
└──────────────┬──────────────────────┘
               ↓
         [Terminal: use node_id for next step]
```

**Output artifact:** Copy top result node_id and code snippet into your current tool/editorspacing/plan.

---

### Mode 2: Cursor-Based Context (IDE Integration)
**When:** Your editor cursor is at file:line and you need surrounding context.

```
┌──────────────────────────────────────┐
│ get_context(file, line)              │
│ Returns: scope + graph neighbors     │
└──────────────┬──────────────────────┘
               ↓
       [Terminal: you have context]
```

**Output artifact:** Paste retrieved context into your refactoring notes or task description.

---

### Mode 3: Graph Traversal (Surgical Dependency Analysis)
**When:** You have a known node_id and need to drill into its callers, callees, or connections.

```
┌──────────────────────────────────────┐
│ find_nodes (locate by name)          │ (optional)
│ Returns: matching node_ids           │
└──────────────┬──────────────────────┘
               ↓
┌──────────────────────────────────────┐
│ read_graph (inspect node)            │

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [senoldogann/LLM-Context-Manager](https://github.com/senoldogann/LLM-Context-Manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
