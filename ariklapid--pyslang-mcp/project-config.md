---
trigger: always_on
description: This file is the handoff document for any AI agent or contributor working in
---

# AGENTS.md

This file is the handoff document for any AI agent or contributor working in
this repository.

Read this first, then read [README.md](./README.md) and
[pyslang-mcp-plan.md](./pyslang-mcp-plan.md).

## Mission

Build a professional, open-source MCP server that gives AI systems
compiler-backed, read-only understanding of Verilog and SystemVerilog projects
through `pyslang`.

The server should be useful to:

- AI coding agents
- LLM-powered IDE tools
- workflow / skill systems
- automation flows that need HDL context

The product is a semantic analysis MCP, not an EDA runtime. Keep the scope
tight and technically honest.

## Current Repo Reality

As of 2026-04-19, this repo has moved past the documentation-only stage into an
alpha local implementation.

What exists:

- `LICENSE`
- `README.md`
- `pyslang-mcp-plan.md`
- this `AGENTS.md`
- `pyproject.toml`
- `src/pyslang_mcp/`
- `tests/` with fixture-backed coverage
- `.github/workflows/ci.yml`
- a local `.venv/` used for research and validation on this machine only

What does not exist yet:

- publish automation
- PyPI release artifacts
- MCP Registry publication
- copy-paste client configuration examples for multiple MCP clients

Do not describe this repo as released or broadly client-ready. A runnable
server now exists, but the project is still alpha and unpublished.

## Product Definition

The intended V1 tools are:

- `pyslang_parse_files`
- `pyslang_parse_filelist`
- `pyslang_get_diagnostics`
- `pyslang_list_design_units`
- `pyslang_describe_design_unit`
- `pyslang_get_hierarchy`
- `pyslang_find_symbol`
- `pyslang_dump_syntax_tree_summary`
- `pyslang_preprocess_files`
- `pyslang_get_project_summary`

V1 non-goals:

- simulation
- synthesis
- waveform viewing
- testbench generation
- RTL editing or refactoring
- remote code execution
- full IDE / language server parity

## Architecture Constraints

These are not optional unless the repo direction is explicitly changed.

- Use the official Python MCP SDK with `FastMCP`.
- Use `stdio` transport first.
- Keep the server strictly read-only.
- Require explicit project roots or clearly client-provided roots.
- Return compact, stable JSON outputs.
- Add output limits and truncation markers early.
- Cache by project config plus file mtimes.
- Prefer a small analysis core with a thin MCP wrapper.

## Progress Done So Far

The main completed work so far is planning, API validation, and a first
end-to-end local implementation.

### Planning

- Repo created on GitHub.
- Initial README written.
- Full implementation plan copied into `pyslang-mcp-plan.md`.

### Local Validation Spike

The following was validated locally in a virtual environment on Python 3.12:

- `pyslang` 10.0.0 installs cleanly on Linux.
- stable Python MCP SDK is the `v1.x` line, not the `main` / v2 pre-alpha docs.
- `mcp` 1.27.0 provides `FastMCP.run("stdio")`, `@mcp.tool()`, and direct
  `call_tool()` testing hooks.

#### `pyslang` APIs confirmed usable

- `pyslang.SyntaxTree.fromFile(path, sourceManager?, bag?)`
- `pyslang.Compilation()`
- `Compilation.addSyntaxTree(tree)`
- `Compilation.getParseDiagnostics()`
- `Compilation.getSemanticDiagnostics()`
- `Compilation.getAllDiagnostics()`
- `Compilation.getDefinitions()`
- `Compilation.getPackages()`
- `Compilation.getRoot()`
- `RootSymbol.topInstances`
- `Symbol.visit(callback)`
- `DiagnosticEngine.reportAll(sourceManager, diagnostics)`
- `DiagnosticEngine.getMessage(code)`
- `DiagnosticEngine.getSeverity(code, location)`

#### Behavior validated with probe files

- single-file parse works
- multi-file package/import flow works
- include directory resolution works
- semantic diagnostics work for unresolved identifiers / types
- hierarchy traversal works through `root.topInstances` plus `visit()`
- instance port connections are available through `InstanceSymbol.portConnections`
- symbol references can be recovered from expressions such as
  `NamedValueExpression.symbol`
- syntax-tree walking works and is suitable for summary generation
- `tree.getIncludeDirectives()` returns useful include metadata

### Implemented Alpha Server

The repository now contains:

- `FastMCP` server wiring in `src/pyslang_mcp/server.py`
- CLI entrypoint in `src/pyslang_mcp/__main__.py`
- project loader with root safety and `.f` parsing
- `pyslang` analysis core for diagnostics, units, hierarchy, symbol search, and
  syntax summaries
- in-memory cache keyed by project config plus tracked file mtimes
- fixture-backed unit, integration, and MCP-level tests
- CI on GitHub Actions for Ubuntu with Python 3.11 and 3.12

The implemented V1 tools match the intended tool list, with one important
honesty constraint:

- `pyslang_preprocess_files` returns preprocessing metadata plus source excerpts, not a
  claimed full standalone-preprocessor output stream

## Important `pyslang` Notes

These findings matter for the implementation.

### 1. Stable MCP SDK target

Use the stable MCP Python SDK behavior and docs, not the in-development `main`
branch docs. Until this project intentionally upgrades, assume v1.x semantics.

### 2. Macro / predefine quirk

`PreprocessorOptions.predefines` behaved correctly when assigned as a complete
list:

```python
pp.predefines = ["EXTRA_MACRO=1"]
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ariklapid/pyslang-mcp](https://github.com/ariklapid/pyslang-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
