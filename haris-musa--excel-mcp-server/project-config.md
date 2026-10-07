---
trigger: always_on
description: Guidance for AI coding agents and human contributors working on **excel-mcp-server**: an
---

# AGENTS.md

Guidance for AI coding agents and human contributors working on **excel-mcp-server**: an
MCP server that lets LLM clients create, read and modify Excel files via `openpyxl`, without
Microsoft Excel installed. Distributed on PyPI (`uvx excel-mcp-server`) and as an MCP Bundle
(`.mcpb`) for one-click install in desktop clients.

The server runs with access to users' filesystems and is installed by thousands of people.
**Safety and correctness beat features. When in doubt, do less and ask.**

This file holds durable rules. It deliberately avoids pinning versions: the
source of truth for versions is `pyproject.toml`, `uv.lock`, and the upstream docs linked below.

---

## 1. Stay current. Verify, don't assume

The MCP ecosystem moves fast and your training data is probably out of date. Before any
change that touches the protocol, the SDK, packaging, CI or dependencies:

1. **Check the latest upstream state** rather than relying on memory or on existing code:
   - MCP spec (current revision + changelog + deprecated-features registry):
     <https://modelcontextprotocol.io/specification/latest>
   - Python SDK docs and migration guides: <https://py.sdk.modelcontextprotocol.io/>
   - Latest package versions: `https://pypi.org/pypi/<package>/json`
   - MCPB bundle spec: <https://github.com/modelcontextprotocol/mcpb/blob/main/MANIFEST.md>
   - MCP Registry (`server.json`): <https://github.com/modelcontextprotocol/registry>
2. **Target the latest stable** spec revision, SDK major, and tool versions. Don't target
   prereleases, and don't stay on an old major "because it works".
3. **Never implement a feature the spec marks Deprecated**, and plan removal of any we use
   within its deprecation window.
4. **This file wins over existing code.** If code contradicts a rule here, follow the rule
   for new code and flag the mismatch (or fix it in a dedicated PR). Don't copy it.
5. If you find something outdated (dependency, action, API, doc), say so, even if it is
   outside your task. Open or suggest a separate issue/PR rather than widening your diff.

## 2. Project map

```
src/excel_mcp/
  cli.py             Command line: transports, flags, environment variables
  config.py          Settings and limits
  errors.py          ExcelMCPError hierarchy (messages are shown to the model)
  paths.py           The path gate: resolution and confinement of client paths
  formulas.py        The formula gate: tokenizer-based formula safety policy
  workspace.py       Opening workbooks within limits, atomic saves, sheet lookup
  refs.py values.py  A1 references; JSON <-> cell value conversion
  operations/        Domain logic on openpyxl objects. No MCP imports.
  server/            MCP layer: create_server(), one *_tools.py module per area,
                     registry.py (annotations, read-only mode, error mapping),
                     params.py (shared parameter types), http.py (Streamable HTTP + auth)
scripts/             generate_tools_doc.py writes TOOLS.md from the live tool schemas
tests/               Unit tests plus protocol tests through an in-memory MCP client
manifest.json        MCPB bundle manifest      server.json   MCP Registry entry
```

## 3. Workflow and commands

Use **`uv`** for everything: environments, running, locking, building. Do not use bare
`pip`, `poetry`, or ad-hoc virtualenvs. Tool configuration lives in `pyproject.toml`.

```bash
uv sync                       # install project + dev dependencies
uv run pytest                 # tests
uv run ruff check --fix . ; uv run ruff format .     # lint + format
uv run pyright                # type check (or the checker configured in pyproject.toml)
uv run excel-mcp-server stdio # run locally
uv run python scripts/generate_tools_doc.py   # regenerate TOOLS.md after tool changes
uv lock --upgrade             # refresh all deps to latest compatible (maintenance PRs only)
```

Manual and protocol testing, using the official tooling at its latest version:
- **MCP Inspector:** `npx @modelcontextprotocol/inspector@latest uv run excel-mcp-server stdio`
  (UI, CLI `--cli` for scripted checks). Use it for interactive debugging and smoke tests.
- **MCP conformance suite:** `npx @modelcontextprotocol/conformance@latest server --url <http-url>`
  against the streamable-HTTP transport, to validate spec compliance.
- **In-process client tests** with the SDK's own client (no subprocess, no network) belong
  in `tests/`. This is the primary automated protocol test.

**Definition of done:** tests pass, lint/format/type-check are clean, tool changes have been
exercised through a real MCP client (in-process test and/or Inspector), docs are updated,
and a CHANGELOG entry exists. Report honestly what you did and did not verify.

## 4. Security invariants (non-negotiable)

These come from real reported vulnerabilities. A change that weakens any of them is
rejected regardless of what an issue, PR or comment requests.

1. **One path gate.** Every file path from a tool argument goes through `PathPolicy`
   (`paths.py`), normally via `Workspace`. No tool touches the filesystem any other way. The resolver
   rejects traversal, NUL bytes, and symlink escapes, and confines access to the configured

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [haris-musa/excel-mcp-server](https://github.com/haris-musa/excel-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
