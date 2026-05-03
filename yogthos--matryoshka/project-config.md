---
trigger: always_on
description: This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.
---

# OpenWolf

@.wolf/OPENWOLF.md

This project uses OpenWolf for context management. Read and follow .wolf/OPENWOLF.md every session. Check .wolf/cerebrum.md before generating code. Check .wolf/anatomy.md before reading files.


# Claude Code Guidelines for Matryoshka RLM

## What This Project Does

**Matryoshka RLM** (Recursive Language Model) processes documents 100x larger than an LLM context window without RAG or chunking. The LLM emits **Nucleus commands** (constrained S-expressions), which are parsed, type-checked, and executed by the **Lattice logic engine**. Results are stored server-side in SQLite — the LLM sees only descriptive handle stubs derived from the command (`$grep_error: Array(1000) [preview...]`), achieving **97%+ token savings**.

```
User Query → LLM Reasons → Nucleus S-expression
                            → Parser validates → Lattice Engine executes
                            → Results stored in SQLite
                            → LLM sees handle stub only
```

## CRITICAL: No Hardcoding

**DO NOT hardcode specific use cases into the prompts or code.**

This is a GENERAL-PURPOSE document analysis tool. When writing prompts or examples:
- Use GENERIC patterns, not domain-specific ones
- Say "data" not "sales data", "values" not "currency values"
- Let the LLM discover the actual data format from the document

## Architecture Overview

### Core Pipeline
- **RLM Entry** (`src/rlm.ts`): Main CLI, document loading, LLM loop orchestration
- **FSM Engine** (`src/fsm/engine.ts`): Generic finite state machine (repl-sandbox)
- **RLM States** (`src/fsm/rlm-states.ts`): QueryLLM → ParseResponse → Execute → Verify → TermOrContinue
- **Adapters** (`src/adapters/`): Model-specific prompting (nucleus, qwen, deepseek)

### Lattice Engine (the core)
- **Parser** (`src/logic/lc-parser.ts`): S-expression lexer → tokens → AST
- **Solver** (`src/logic/lc-solver.ts`): Term evaluator using miniKanren for relational reasoning
- **Type Inference** (`src/logic/type-inference.ts`): Type checking before execution
- **Search** (`src/logic/bm25.ts`, `semantic.ts`, `rrf.ts`): BM25, TF-IDF, Reciprocal Rank Fusion
- **Q-value Reranker** (`src/logic/qvalue.ts`): Learns across turns

### Handle System (97% token savings)
- **SessionDB** (`src/persistence/session-db.ts`): In-memory SQLite with FTS5
- **HandleRegistry** (`src/persistence/handle-registry.ts`): Create/manage handle references
- **HandleOps** (`src/persistence/handle-ops.ts`): Server-side filter/map/count/sum on handles
- **HandleSession** (`src/engine/handle-session.ts`): Wraps NucleusEngine + handle storage

### Nucleus Engine
- **NucleusEngine** (`src/engine/nucleus-engine.ts`): Standalone command executor, bindings, cross-query state
- **Bindings**: `RESULTS` (latest array), `_1`/`_2`/... (per-turn), `_fn_name` (synthesized fns)

### Synthesis (Barliman-style)
- **Coordinator** (`src/synthesis/coordinator.ts`): Routes to regex/extractor/relational synthesizers
- **Evalo DSL** (`src/synthesis/evalo/`): Type-checked extraction language
- **miniKanren** (`src/minikanren/`): Relational programming engine for constraint solving
- LLM provides CONSTRAINTS (input/output examples), NOT code — synthesizer builds programs

### Code Intelligence
- **Tree-sitter** (`src/treesitter/`): Symbol extraction (functions, classes, methods) for .ts/.js/.py/.go/.md
- **Symbol Graph** (`src/graph/`): Knowledge graph — callers, callees, ancestors, descendants, implementations

### Entry Points / Adapters
- `src/lattice-mcp-server.ts` — MCP server for Claude Code (handle-based)
- `src/mcp-server.ts` — MCP server with full LLM orchestration
- `src/tool/adapters/pipe.ts` — JSON subprocess control (stdin/stdout)
- `src/tool/adapters/http.ts` — REST API with session lifecycle
- `src/tool/adapters/claude-code.ts` — Auto-registers as MCP tools

### Session Management
- Session timeout: **10 minutes** inactivity (configurable in `lattice-mcp-server.ts`)
- Timer resets on every query
- Single session per MCP instance
- Max document: 50MB, max handles: 200 (LRU eviction), max grep matches: 10,000

## Key Principle: Barliman-Style Synthesis

The LLM provides CONSTRAINTS (input/output examples), NOT code implementations.
The synthesizer builds programs automatically from examples.

## Using Nucleus for Large File Analysis

When you need to analyze files larger than ~500 lines, use the Nucleus tool instead of reading files directly. This saves 80%+ tokens.

### Recommended Workflow for Codebase Analysis
1. **Use Glob first** to discover all relevant files (e.g., `**/*.py`, `**/*.ts`)
2. **Read small files directly** (<300 lines) - Nucleus is overkill for these
3. **Use Nucleus only for large files** (>500 lines)
4. **Aggregate data across ALL files**, not just the largest one

This workflow ensures complete analysis. Using Nucleus alone misses:
- Small config/utility files with important details
- Multi-file patterns (imports, classes across files)
- File discovery and project structure

### When to Use Nucleus
- File is >500 lines
- You need multiple searches on the same file
- You're extracting or aggregating structured data

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yogthos/Matryoshka](https://github.com/yogthos/Matryoshka) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
