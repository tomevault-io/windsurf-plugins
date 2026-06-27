---
trigger: always_on
description: graphlens/                      ← uv workspace root (also the core library)
---

# graphlens — Architecture Guide

## Project layout

```
graphlens/                      ← uv workspace root (also the core library)
  src/graphlens/                ← core: models, contracts, registry, exceptions, utils
  packages/
    graphlens-python/           ← Python language adapter
    graphlens-typescript/       ← TypeScript language adapter
    graphlens-go/               ← Go language adapter (structure + go.mod)
    graphlens-rust/             ← Rust language adapter (structure + Cargo.toml)
    graphlens-php/              ← PHP language adapter (tree-sitter + PHPantom)
    graphlens-cli/              ← CLI: analyze / query / visualize / neo4j
  tests/                         ← core library tests
  examples/                      ← standalone usage examples (no CLI dep)
    demo_resolved_graph.py      ← Python: print node/edge stats + find-usages
    demo_resolved_graph_ts.py   ← TypeScript: same, via TypeScript Compiler API
    visualize_graph.py          ← standalone HTML graph viewer (vis.js)
    neo4j_export.py             ← standalone Neo4j export script
```

All packages use the `src/` layout and `uv_build` as build backend.

Each language adapter follows this internal layout:
```
packages/graphlens-<lang>/
  src/graphlens_<lang>/
    __init__.py              ← exports <Lang>Adapter (+ <Lang>Resolver if public)
    _adapter.py              ← LanguageAdapter subclass + _analyze_root()
    _visitor.py              ← ASTVisitor + ImportClassifier + OccurrenceRef
    _resolver.py             ← SymbolResolver subclass (e.g. TyResolver)
    _deps.py                 ← DependencyFileParser implementations + default list
    _project_detector.py     ← is_<lang>_project(), find_<lang>_roots(), detect_project_name()
    _module_resolver.py      ← file→qualified_name, source root detection
```

---

## Core principles

### 1. Adapters are pure data producers
An adapter parses source files and returns a `GraphLens`. It never writes to
any backend, database, or file system. The graph is the only output.

### 2. graphlens core is minimal
`core` lives at the workspace root under `src/graphlens/`. It contains only:
models, contracts (ABCs), registry, exceptions, utils.
No pipeline, no orchestration, no I/O. Orchestration belongs in a separate
package or in user code.

**Non-goals (explicit scope boundary).** graphlens produces a graph IR and
stops there. It does NOT: persist state or own a database (backends are a
separate consuming layer); watch the filesystem or re-index incrementally on
its own (scans are pure; deterministic IDs enable, but the caller drives,
incremental updates); compute embeddings, semantic search, or relevance
ranking; provide a UI or an agent runtime (`visualize` emits static HTML,
`mcp` exposes query tools — neither hosts a long-running service). These belong
to tools built on top of graphlens. See `website/docs/intro.md` → "Scope &
Non-goals".

### 3. SQLAlchemy dialect pattern for adapters
Adapters register themselves via `importlib.metadata` entry points:

```toml
# In the adapter's pyproject.toml
[project.entry-points."graphlens.adapters"]
python = "graphlens_python:PythonAdapter"
```

Callers resolve adapters through the registry — no direct imports needed:

```python
from graphlens import adapter_registry
adapter = adapter_registry.load("python")()
graph = adapter.analyze(project_root)
```

### 4. Tree-sitter + type-aware resolver
Every adapter uses Tree-sitter for structure extraction, occurrence roles
(call/read/write/annotation/base), and spans. A language-specific
`SymbolResolver` handles type-aware resolution — mapping occurrence positions
to definition nodes and emitting CALLS/REFERENCES/HAS_TYPE/INHERITS_FROM
edges. Tree-sitter is no longer the sole engine; it hands off precise position
data that the resolver consumes.

The Python adapter's `TyResolver` spawns a `ty server` LSP subprocess (Astral
ty, Rust-based); files are opened lazily on the first query per file, and
`open_file()` drains `publishDiagnostics` before returning so definition
queries never block on background analysis. The TypeScript
adapter's `TsResolver` is a Node-subprocess Compiler-API resolver that batches
all occurrence queries into a single `resolve_all` call to a bundled
`ts_resolver.js` script, installing typescript on-demand into a cache dir.

Parser setup (one module-level singleton per adapter):
```python
import tree_sitter_<lang> as ts_lang
from tree_sitter import Language, Parser, Node as TSNode

_LANGUAGE = Language(ts_lang.language())
_parser = Parser(_LANGUAGE)

def parse_<lang>(source: bytes) -> tree_sitter.Tree:
    return _parser.parse(source)
```

### 5. Visitor pattern: dispatch by node.type
```python
class <Lang>ASTVisitor:
    def visit(self, node: TSNode) -> None:
        handler = getattr(self, f"_visit_{node.type}", None)
        if handler:
            handler(node)
        else:
            self._visit_children(node)

    def _visit_children(self, node: TSNode) -> None:
        for child in node.children:
            self.visit(child)
```

All state lives on three stacks pushed/popped as scope changes:
- `_scope_stack: list[str]` — qualified name prefix
- `_container_stack: list[str]` — current parent node ID

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Neko1313/graphlens](https://github.com/Neko1313/graphlens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
