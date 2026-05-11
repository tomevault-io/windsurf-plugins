---
trigger: always_on
description: Guidance for Claude Code (and similar coding agents) working on this repo.
---

# CLAUDE.md

Guidance for Claude Code (and similar coding agents) working on this repo.

## Project summary

**codegraph** (package: `codegraph`) is a Python tool that indexes TypeScript and Python codebases into a Neo4j property graph. It walks source with tree-sitter, recognises framework constructs (NestJS controllers / injectables / modules, React components and hooks, TypeORM entities, GraphQL operations, Python classes and decorators), and loads typed nodes + edges into Neo4j. Downstream consumers:

- **CLI**: `codegraph index`, `codegraph query`, `codegraph validate`, `codegraph wipe` — Typer app.
- **MCP server**: `codegraph-mcp` stdio server with 16 read-only tools. Optional extra (`pip install "codegraph[mcp]"`).
- **REPL**: interactive Cypher shell at `codegraph repl`.

This repo is itself **Python**, and codegraph parses Python since Stage 1 shipped. So we can dogfood: Claude Code can query the graph of codegraph-the-codebase while implementing codegraph-the-tool.

## Using the graph during development

Two slash commands wire the local Neo4j graph into your Claude Code workflow. **Use them** — they exist to catch mistakes the language server can't.

### `/graph <cypher>` — query the live graph

Read-only Cypher against `bolt://localhost:7688`. Prefer this over manual `codegraph query` in Bash because the slash command has the permissions pre-approved and a body of canonical query patterns. See `.claude/commands/graph.md` for examples.

**When to run `/graph`**:

- **Before renaming** a class, function, or file → check blast radius with `MATCH (f:File)-[r:IMPORTS_SYMBOL]->(g:File) WHERE r.symbol = 'X' RETURN f.path, g.path`.
- **Before deleting** a function or method → confirm nothing depends on it.
- **Before moving** a module → count incoming IMPORTS edges; you'll need to update every caller.
- **When asked "who calls X?" or "where is Y used?"** — almost always a one-query answer.
- **When in doubt about what exists** — `/graph "MATCH (c:Class) WHERE c.name CONTAINS 'Foo' RETURN c.name, c.file"` is faster and more exhaustive than grep.

### `/graph-refresh` — update the graph

Re-indexes this repo's Python package + its tests (`codegraph/codegraph/` + `codegraph/tests/`) so `/graph` queries reflect the latest on-disk state. **Run it after any structural edit** — adding, removing, renaming, or moving classes / functions / methods / imports / decorators. Cosmetic edits don't need a refresh.

Takes ~5 seconds. Uses `--no-wipe` so other indexed graphs (e.g. Twenty) survive. See `.claude/commands/graph-refresh.md`.

### Daily power-tool commands

Five purpose-built wrappers over `/graph` for the patterns you'll reach for most often. Each is a thin shell around a canonical Cypher query — run them before changing code, not after.

| Command | Use case |
|---|---|
| `/blast-radius <Symbol>` | Before renaming / deleting / moving a class, function, or method — see every caller, subclass, DI consumer, and importer |
| `/dead-code [path_prefix]` | Sweep for orphan functions, classes, atoms, endpoints. Framework entry points (`@mcp.tool()`, `@app.command()`, `@pytest.fixture`) are excluded automatically |
| `/who-owns <path>` | Latest author + top-5 contributors + CODEOWNERS team for a file. Requires an ownership-aware index pass (not `--skip-ownership`) |
| `/trace-endpoint <url_substring>` | Endpoint → handler method → every method reachable within 4 `CALLS` hops. Good for impact analysis + security review |
| `/arch-check` | Built-in conformance policies: import cycles, cross-package violations, Controller→Repository bypass. Fork the command to add project-specific policies |

### Architecture drift (CI gate)

`codegraph arch-check` is a first-class CLI subcommand that runs the same three built-in policies as `/arch-check` and exits non-zero on any violation. `.github/workflows/arch-check.yml` wires it into CI: on every PR to `main`, GitHub Actions spins up `neo4j:5.24-community` as a service container, indexes the repo with `codegraph index`, runs the check, and uploads the JSON report as a build artifact. Any violation blocks the merge.

**Configuring the trigger scope**: edit the `on:` block at the top of `.github/workflows/arch-check.yml`. The shipped default is `pull_request: branches: [main]` (strictest, lowest-noise). Two commented-out alternatives sit right below it: `push: branches: [dev]` to catch drift the moment it lands, and `workflow_dispatch` for manual runs from the Actions UI. Uncomment what you need.

**Reproducing a failing check locally**: `cd codegraph && codegraph index . -p codegraph -p tests --skip-ownership && codegraph arch-check`. Exit code mirrors CI.

Policy reference + false-positive guidance: `codegraph/docs/arch-policies.md`.

### What's indexed (and what isn't)

<!-- codegraph:stats-begin -->
~20 files, 56 classes, 134 module functions, ~180 methods
<!-- codegraph:stats-end -->

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cognitx-leyton/codegraph](https://github.com/cognitx-leyton/codegraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-01 -->
