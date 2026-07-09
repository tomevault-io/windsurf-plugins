---
trigger: always_on
description: Repo-specific invariants for this MCP server. These are **self-enforcing reminders**: follow them without waiting to be asked. (General working preferences live in the user-global `~/.claude/CLAUDE.md`.)
---

# automox-mcp — project instructions

Repo-specific invariants for this MCP server. These are **self-enforcing reminders**: follow them without waiting to be asked. (General working preferences live in the user-global `~/.claude/CLAUDE.md`.)

## Adding / removing / renaming a tool

A tool's name and count appear in several hand-maintained places. When you change the tool set, update **all** of these in the same change — CI guards some, but not all:

| Location | What to update | CI-guarded? |
|---|---|---|
| `src/automox_mcp/tools/<domain>_tools.py` | the `@server.tool` registration | — |
| `src/automox_mcp/tools/meta_tools.py` `_DOMAIN_CATALOG` | the `(name, description)` entry (the model-facing discovery directory) | ✅ `test_doc_tool_counts.py` |
| `docs/tool-reference.md` | the bullet **and** its `## Domain (N tools)` section count **and** the top "all N tools" header | ✅ `test_doc_tool_counts.py` |
| `README.md` + `mcpb/manifest.json` | total / read / write counts | ✅ `test_doc_tool_counts.py` |
| `mcpb/manifest.json` `tools[]` (+ `prompts[]` on prompt changes) | **regenerate, don't hand-edit**: `uv run python scripts/generate_mcpb_catalog.py` (feeds Claude Desktop's "Tools"/"Prompts" details sections) | ✅ `test_doc_tool_counts.py` |
| `docs/api-coverage.md` | coverage map / omission rationale / build-backlog rows | ❌ **manual — easy to forget** |
| `tests/smoke_production.py` | live read-side coverage for new read tools (not destructive writes) | ❌ **manual, not in CI** |
| `CHANGELOG.md` | the entry under the active version | ❌ manual |

The current split is **133 tools / 85 read / 48 write**. `discover_capabilities` is intentionally excluded from `_DOMAIN_CATALOG`.

## Adding / removing an MCP resource or MCP App

Resources (incl. `ui://` MCP App UIs) and the Apps extension have their own hand-maintained surfaces — and unlike tools, **none of them are CI-guarded**:

| Location | What to update | CI-guarded? |
|---|---|---|
| `src/automox_mcp/resources/<name>_resources.py` + `resources/__init__.py` | the `@server.resource` registration + its `register_*` call | — |
| `docs/tool-reference.md` + `mcpb/manifest.json` | the **"N MCP resources"** count | ✅ `test_doc_tool_counts.py` |
| `docs/tool-reference.md` `## MCP Resources` | the resource-table **row** and (for Apps) the `### MCP Apps` note | ❌ **manual — easy to forget** |
| `CHANGELOG.md` | the entry under the active version | ❌ manual |

The **"N MCP resources"** count in `docs/tool-reference.md` and `mcpb/manifest.json` is now **CI-guarded** by `tests/test_doc_tool_counts.py` (real-FastMCP resource introspection, mirroring the tool guard). ⚠️ The resource-table **row** and the `### MCP Apps` note are still **manual** — a new resource added without its row/note ships silently. There are currently **14** resources (9 reference + 5 `ui://` MCP App UIs).

## Advertising `output_schema` (demand-driven, not blanket)

Advertise `output_schema` on a read tool only when it becomes a **render target** — an App/UI surface, a schema-aware host that validates output, or the model needing the typed shape to chain calls. Do **not** speculatively schematize the read surface: `maybe_format_markdown` already emits `structuredContent` (issue #177), so un-schematized read tools are fully consumable. A blanket schema is either dead weight (a permissive shape that documents nothing) or — across heterogeneous tools — an unverifiable contract: FastMCP **validates returns against the schema at runtime**, smoke isn't in CI, and a too-strict schema that drifts from the live payload fails the tool in production (cf. the #132 live-contract bugs). When you do add one, it must validate the **real** `structuredContent` payload, smoke-verified, and the model must be permissive (all-optional, `dict[str, Any]` for variable sub-objects, never `extra="forbid"`). Rationale and the per-tool decisions live in `docs/api-coverage.md`.

MCP Apps specifics: an App is a `ui://` HTML resource (FastMCP auto-resolves the `text/html;profile=mcp-app` MIME) plus an `app=AppConfig(...)` on the entry tool's `@server.tool` (top-level `FastMCP.tool` accepts `app=`; the lower-level provider decorator does not). Keep App UIs **self-contained** (inline JS/CSS, no CDN imports) so they run under the host's default deny-all CSP with no `ResourceCSP` domains. `prefab_ui` is not installed — do not use `PrefabAppConfig`/`@app.ui()`.

## Local gate before every push

A **pre-push git hook is the deterministic backstop**: `.pre-commit-config.yaml` runs the whole-repo gate at `pre-push`, so a push can't reach origin without passing it — no need to remember to run it by hand. After a fresh clone, activate it once:

```
uv run pre-commit install        # wires both pre-commit and pre-push hooks
```

The pre-push gate (and the manual equivalent) is:

```
uv run ruff format --check .     # NB: separate tool from `ruff check`
uv run ruff check .
uv run mypy .
uv run pytest --cov=automox_mcp --cov-fail-under=90
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AutomoxCommunity/automox-mcp](https://github.com/AutomoxCommunity/automox-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
