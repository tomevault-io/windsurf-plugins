---
trigger: always_on
description: Chiasmus is an MCP (Model Context Protocol) server that gives LLMs access to formal verification via **Z3** (SMT solver) and **Tau Prolog**. It also provides tree-sitter-based source code call graph analysis. The server translates natural language problems into formal logic using a template-based pipeline, verifies results with mathematical certainty, and analyzes call graphs for reachability, dead code, and impact analysis.
---

# AGENTS.md — Chiasmus

## Project Overview

Chiasmus is an MCP (Model Context Protocol) server that gives LLMs access to formal verification via **Z3** (SMT solver) and **Tau Prolog**. It also provides tree-sitter-based source code call graph analysis. The server translates natural language problems into formal logic using a template-based pipeline, verifies results with mathematical certainty, and analyzes call graphs for reachability, dead code, and impact analysis.

- **Language**: TypeScript (strict mode, ESM)
- **Runtime**: Node.js ≥20
- **License**: Apache-2.0
- **Package**: npm package `chiasmus`, exposes a CLI binary

## Commands

```bash
npm run build          # Compile TypeScript (tsc) → dist/
npm run typecheck      # Type-check only (tsc --noEmit)
npm run test           # Run tests in watch mode (vitest)
npm run test:run       # Run tests once (vitest run)
npm run test:coverage  # Run tests with V8 coverage report
npm run mcp            # Run MCP server locally via tsx
```

### CI Pipeline

CI runs on push/PR to `main` (`.github/workflows/test.yml`):
1. `npm ci`
2. `npm run typecheck`
3. `npm run test:run`

Tested on Node 20 and 22.

## Code Organization

```
src/
├── mcp-server.ts          # Entry point — MCP server, all tool handlers, CLI bootstrap
├── config.ts              # Loads ~/.chiasmus/config.json
├── solvers/
│   ├── types.ts           # SolverType, SolverResult (discriminated union), Solver, SolverInput
│   ├── session.ts         # SolverSession — factory for Z3/Prolog solver instances
│   ├── z3-solver.ts       # Z3 WASM wrapper (z3-solver npm), auto-strips check-sat/get-model
│   ├── prolog-solver.ts   # Tau Prolog wrapper, derivation tracing via assertz
│   └── correction-loop.ts # Bounded repair loop (delegates to repl-sandbox)
├── formalize/
│   ├── engine.ts          # FormalizationEngine — template selection, LLM slot-filling, solve pipeline
│   ├── validate.ts        # lintSpec — structural validation, auto-fixes for SMT-LIB/Prolog
│   └── feedback.ts        # classifyFeedback — converts SolverResult to human-readable feedback
├── skills/
│   ├── types.ts           # SkillTemplate, SlotDef, Normalization, SkillMetadata
│   ├── library.ts         # SkillLibrary — SQLite-backed template storage, BM25 search
│   ├── starters.ts        # 8 built-in starter templates (5 Z3, 3 Prolog)
│   ├── bm25.ts            # BM25 keyword search for template retrieval
│   ├── craft.ts           # craftTemplate — user-created template validation + storage
│   ├── learner.ts         # SkillLearner — LLM-driven template extraction from solutions
│   └── relationships.ts   # Template relationship/suggestion graph
├── graph/
│   ├── types.ts           # CodeGraph, LanguageAdapter, FileNode, Hyperedge, DefinesFact, CallsFact.calleeQN, ImportsFact.resolved, FileTypeInfo
│   ├── parser.ts          # tree-sitter lang registry + sync/async parse for TS/JS/Py/Go/Clojure
│   ├── extractor.ts       # AST walking + per-language call graph extraction + TS/JS collectTypeInfo
│   ├── facts.ts           # graphToProlog — CodeGraph → Prolog facts (incl. calls_qn/3, imports_resolved/3)
│   ├── analyses.ts        # runAnalysis — dispatches all graph analyses
│   ├── native-analyses.ts # O(V+E) cycles/reachability/impact/dead-code/callers/callees
│   ├── graph-util.ts      # Shared helpers: buildUndirectedGraph, forEachUndirectedEdge, undirectedDegree
│   ├── community.ts       # Louvain community detection + cohesion score
│   ├── insights.ts        # detectHubs, detectBridges, detectSurprisingConnections
│   ├── diff.ts            # graphDiff — set diff on nodes + (src,tgt) edge keys
│   ├── entry-points.ts    # Heuristic entry-point detection (zero-in-degree exports)
│   ├── cache.ts           # SHA256 per-file cache + LRU eviction + named snapshots (proper-lockfile)
│   ├── mermaid.ts         # parseMermaid — Mermaid flowcharts/state diagrams → Prolog facts
│   ├── type-env.ts        # TS/JS three-tier type inference + class field/method extraction
│   ├── resolve-calls.ts   # Project-wide QN resolution: inheritance-aware field/method registry
│   ├── tsconfig-aliases.ts # Parses tsconfig.json paths/baseUrl with JSONC + extends chain
│   ├── suffix-index.ts    # Suffix index over batch files for relative import resolution
│   └── adapter-registry.ts # Auto-discovery of chiasmus-adapter-* npm packages
├── search/
│   ├── engine.ts          # buildSearchCorpus + runSearch — cosine-sim code search
│   ├── vector-store.ts    # In-process linear-scan store; serializable
│   ├── embedding-cache.ts # SHA-256-keyed persistent cache, atomic writes
│   └── index.ts           # Module exports
├── llm/
│   ├── types.ts           # LLMAdapter + EmbeddingAdapter interfaces, LLMMessage
│   ├── anthropic.ts       # Anthropic/DeepSeek/OpenAI provider factories (createLLMFromEnv, createEmbeddingFromEnv)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yogthos/chiasmus](https://github.com/yogthos/chiasmus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
