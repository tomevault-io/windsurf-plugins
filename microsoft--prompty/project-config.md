---
trigger: always_on
description: Prompty is being rebuilt from the ground up. The type system that originated in Prompty's TypeSpec specification has been extracted into [AgentSchema](https://microsoft.github.io/AgentSchema/) as the canonical, lower-level type layer. Prompty v2 takes a dependency on the `agentschema` Python package and builds its runtime on top.
---

# Prompty v2 — Complete Rebuild Plan

## Overview

Prompty is being rebuilt from the ground up. The type system that originated in Prompty's TypeSpec specification has been extracted into [AgentSchema](https://microsoft.github.io/AgentSchema/) as the canonical, lower-level type layer. Prompty v2 takes a dependency on the `agentschema` Python package and builds its runtime on top.

**Core principle**: Prompty is a markdown file format (`.prompty`) for LLM prompts. The frontmatter uses AgentSchema's PromptAgent schema. The markdown body becomes `instructions`. The runtime loads, renders, parses, and executes prompts.

### What Prompty Uses from AgentSchema

Only the PromptAgent subset — not the full spec:

| AgentSchema Type                                           | Prompty Uses                                        |
| ---------------------------------------------------------- | --------------------------------------------------- |
| `PromptAgent`                                              | Yes — the loaded `.prompty` file becomes this       |
| `Model`                                                    | Yes — model configuration                           |
| `Connection` (all subtypes)                                | Yes — auth/endpoint config                          |
| `ModelOptions`                                             | Yes — temperature, maxOutputTokens, etc.            |
| `Template` / `Format` / `Parser`                           | Yes — template engine config                        |
| `PropertySchema` / `Property`                              | Yes — inputSchema/outputSchema                      |
| `FunctionTool`                                             | Yes — local function calling                        |
| `McpTool`                                                  | Yes — MCP server integration                        |
| `OpenApiTool`                                              | Yes — API-based tools                               |
| `CustomTool`                                               | Yes — wildcard `*` catch-all for unknown tool kinds |
| `Workflow`                                                 | No                                                  |
| `ContainerAgent`                                           | No                                                  |
| `AgentManifest`                                            | No                                                  |
| `WebSearchTool` / `FileSearchTool` / `CodeInterpreterTool` | No                                                  |

### Key Design Decisions

1. **`agentschema` is a pip dependency** — no code generation, no TypeSpec in Prompty repo
2. **No `kind` in `.prompty` files** — the loader injects `kind: "prompt"` since Prompty only produces PromptAgents
3. **Markdown body → `instructions`** — the loader splits frontmatter/body and sets `agent.instructions = body`
4. **Legacy migration in the loader** — old-format `.prompty` files still load with deprecation warnings
5. **Invoker pattern retained** — Renderer → Parser → Executor → Processor pipeline, redesigned in Phase 2
6. **Use `uv` exclusively** for all Python environment and package management. Never use `pip`, `pip install`, `python -m pip`, or `python -m venv` directly. Use `uv venv`, `uv pip install`, `uv run`, etc. instead.
7. **Use `AgentDefinition.load(data, ctx)`** — not `PromptAgent.load()`. `AgentDefinition.load()` dispatches on `kind` via `load_kind()` to create a `PromptAgent`, then populates base fields (`name`, `metadata`, `inputSchema`, `outputSchema`, `description`, `displayName`). `PromptAgent.load()` only handles prompt-specific fields (`model`, `tools`, `template`, `instructions`)
8. **`Property` has `default` and `example`** — not `value`. The `value` field mentioned in early specs does not exist in agentschema. Use `default` for sample/default values
9. **`${protocol:value}` expansion via `LoadContext.pre_process`** — agentschema's `LoadContext(pre_process=fn)` walks every dict in the tree, calling `fn(dict)` → mutated dict. This is the ideal hook for resolving `${env:VAR}`, `${file:path}`, etc. without a separate recursive pass

---

## Python Coding Rules

### Environment & Tooling

- **Python ≥ 3.11** — use modern syntax: `X | Y` unions, `match`/`case`, `ExceptionGroup`, `type` aliases.
- **`uv` exclusively** — never use `pip`, `pip install`, `python -m pip`, or `python -m venv`. Use `uv venv`, `uv pip install`, `uv run`, etc. Package installs: `uv pip install -e ".[dev,all]"`.
- **Flit build system** — `flit_core >=3.11,<4`. Version is dynamic via `_version.py`.
- **Ruff** for linting and formatting. Config lives in `pyproject.toml`:
  - Line length: **120**
  - Target: `py311`
  - Rules: `E`, `F`, `I` (isort), `UP` (pyupgrade)
  - `E501` (line too long) is **ignored** (handled by formatter)
  - `__init__.py` files ignore `F401` (unused imports — they're re-exports)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/prompty](https://github.com/microsoft/prompty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
