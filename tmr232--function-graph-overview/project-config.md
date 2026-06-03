---
trigger: always_on
description: Function-Graph-Overview generates control-flow graphs (CFGs) from source code. It runs as a VSCode extension and a web demo. It uses **tree-sitter** WASM parsers to produce ASTs, then walks them with language-specific statement handlers to build CFGs.
---

# AGENTS.md

## Project Overview

Function-Graph-Overview generates control-flow graphs (CFGs) from source code. It runs as a VSCode extension and a web demo. It uses **tree-sitter** WASM parsers to produce ASTs, then walks them with language-specific statement handlers to build CFGs.

## Commands

- **Install**: `bun install`
- **Test**: `bun vitest run`
- **Lint & format**: `bun lint`
- **Dev server (web demo)**: `bun demo`
- **Generate parsers**: `bun generate-parsers`
- **Build**: `bun run build` (check package.json for exact scripts)

## Platform Notes

This project uses **bun** as its JavaScript runtime, package manager, and script runner — not npm or node. Use `bun` in place of `npm run`, `npx`, and `node` for all commands (e.g., `bun install`, `bun vitest run`, `bun run build`).

On Windows, bun runs commands via PowerShell. Unix utilities like `tail`, `head`, `grep` are not available — use PowerShell equivalents or omit them. Pipe bun command output directly without filters (e.g., `bun vitest run 2>&1` not `bun vitest run 2>&1 | tail -30`).

## Supported Languages

C, C++, Go, Java, Python, TypeScript, TSX.

Each language has a `cfg-<lang>.ts` file in `src/control-flow/`.

## Architecture: How CFGs Are Built

### Core Types (src/control-flow/cfg-defs.ts)

- **`BasicBlock`** — The fundamental unit of CFG construction. Has `entry` (node ID), `exit` (node ID or null), plus optional `breaks`, `continues`, `gotos`, `labels`, `functionExits`.
- **`CFG`** — Final result: `{ graph, entry, offsetToNode }`.
- **`CFGGraph`** — A `MultiDirectedGraph<GraphNode, GraphEdge>` from graphology.
- **`GraphNode`** — Has `type: NodeType`, `code`, `lines`, `markers`, `cluster`, `targets`, `startOffset`.
- **`NodeType`** — One of: `START`, `END`, `STATEMENT`, `CONDITION`, `RETURN`, `BREAK`, `CONTINUE`, `MERGE`, `LOOP_HEAD`, `FOR_EXIT`, `SWITCH_CONDITION`, `CASE_CONDITION`, `SWITCH_MERGE`, `THROW`, `YIELD`, `GOTO`, `LABEL`, `EMPTY`, etc.
- **`EdgeType`** — `regular`, `consequence`, `alternative`, `exception`.
- **`BlockHandler`** — Collects breaks/continues/gotos/functionExits from child blocks and provides `forEachBreak`, `forEachContinue`, `forEachFunctionExit`, `processGotos` methods.
- **`BuilderOptions`** — `{ flatSwitch?, markerPattern?, callProcessor? }`.

### GenericCFGBuilder (src/control-flow/generic-cfg-builder.ts)

The shared engine all languages use. Constructed with `StatementHandlers` + `BuilderOptions`.

- **`StatementHandlers`** — `{ named: { [nodeType: string]: StatementHandler }, default: StatementHandler }`. Maps tree-sitter AST node type names to handler functions.
- **`StatementHandler`** — `(syntax: SyntaxNode, ctx: Context) => BasicBlock`. Converts one AST node into a CFG fragment.
- **`Context`** — `{ builder, options, matcher, dispatch, state, link, extra?, callProcessor? }`. Passed to every handler.
  - `builder` (Builder) — Adds nodes/edges to the graph.
  - `matcher` (BlockMatcher) — Runs tree-sitter queries on AST nodes to extract sub-nodes.
  - `dispatch.single(syntax)` / `dispatch.many(statements, parent)` — Recursively process child AST nodes.
  - `state` (BlockHandler) — Tracks breaks/continues/gotos for the current scope.
  - `link.syntaxToNode()` / `link.offsetToSyntax()` — Maps source offsets to CFG nodes.

**`buildCFG(functionNode)`** flow:
1. Creates a `START` node.
2. Gets `functionNode.childForFieldName("body")`.
3. Calls `dispatchMany` on the body's named children.
4. Resolves gotos, creates implicit `RETURN` node, links edges.

**`dispatchMany(statements)`** — Filters out non-marker comments, calls `dispatchSingle` per statement, chains blocks (exit→entry edges).

**`dispatchSingle(syntax)`** — Looks up `handlers.named[syntax.type]` or falls back to `handlers.default`.

### LanguageDefinition (src/control-flow/cfg.ts)

Each language exports a `LanguageDefinition`:
```typescript
{
  wasmPath: string;              // import of parsers/tree-sitter-<lang>.wasm?url
  createCFGBuilder: (options: BuilderOptions) => CFGBuilder;
  functionNodeTypes: string[];   // AST types representing functions
  extractFunctionName: (node: SyntaxNode) => string | undefined;
}
```

Registered in `languageDefinitions: Record<Language, LanguageDefinition>` in `src/control-flow/cfg.ts`.

### Common Patterns (src/control-flow/common-patterns.ts)

Reusable handler factories for common control-flow constructs:

- **`cStyleIfProcessor(query)`** — C/C++/TS-style if/else-if/else chains.
- **`cStyleForStatementProcessor(query)`** — C-style `for(init;cond;update)` loops.
- **`cStyleWhileProcessor()`** — `while(cond) body` loops.
- **`cStyleDoWhileProcessor()`** — `do body while(cond)` loops.
- **`forEachLoopProcessor(definition)`** — Range/for-each loops (Go `for range`, Python `for in`, TS `for of`).
- **`labeledBreakProcessor(query)` / `labeledContinueProcessor(query)`** — Break/continue with optional labels.
- **`processReturnStatement`**, **`processBreakStatement`**, **`processContinueStatement`** — Simple flow-exit handlers.
- **`processGotoStatement`**, **`processLabeledStatement`** — Goto/label support.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmr232/function-graph-overview](https://github.com/tmr232/function-graph-overview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
