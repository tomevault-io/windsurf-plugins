---
trigger: always_on
description: Guidance for Claude Code when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

**mcp-scribe** turns an OpenAPI schema into a production-grade MCP server. It is a
compiler with a runtime attached: it lowers an OpenAPI document into MCP tools, then
executes those tools as real HTTP requests with auth, retries, rate limiting, and
response shaping already handled.

Package lives in `src/mcp_scribe/` (src layout, Poetry, `packages = [{include = "mcp_scribe", from = "src"}]`).

## Commands

```bash
poetry install --all-extras --with lint,test

poetry run pytest -q                        # 201 tests, ~1s
poetry run pytest tests/test_toolset.py -q  # one file
poetry run pytest -m remote                 # network tests (opt-in marker)

poetry run ruff check src tests             # lint
poetry run black src tests examples         # format  <-- see below
poetry run mypy src/mcp_scribe              # types
poetry build
```

### Formatting: black only. Do not run `ruff format`.

**black at `line-length = 70` is the formatter of record.** `[tool.black]` and
`[tool.ruff]` both pin 70 in `pyproject.toml`, and they must stay identical — when they
drift, each tool rewrites what the other just wrote and every file shows up as modified.

At 70 columns black and `ruff format` genuinely disagree (tuple-unpacking RHS,
`assert x, msg` parenthesization). They agreed at 100; they do not at 70. So:

- **Format with `black`.** Never run `ruff format` — it will fight black on ~2 files and
  reintroduce the churn loop.
- **Lint with `ruff check`.** That is ruff's only job here.
- `E501` is in ruff's `ignore` list. At 70 the remaining long lines are URLs, OpenAPI
  keyword names, and string literals no formatter can split; enforcing it produces 400+
  findings black cannot act on.

If a `ruff check --fix` result and black disagree, verify they reach a fixpoint (apply
the fix, run black, confirm nothing changes) before committing.

> `README.md`'s Development section still lists `ruff format --check src tests`. That
> command now fails by design — it should be `black --check`.

## Architecture

The pipeline is one directional flow. Each stage hands the next a narrower type.

```
spec/loader.py  ->  spec/parser.py  ->  toolset.py  ->  server.py  ->  runtime/executor.py
   fetch              normalize          lower          serve           execute
```

| Module | Responsibility |
| --- | --- |
| `spec/loader.py` | Fetch from URL / file / stdin. JSON or YAML. Caching, size limits. |
| `spec/refs.py` | Collect external `$ref` documents in one pass, load concurrently. |
| `spec/swagger2.py` | Convert Swagger 2.0 up to OpenAPI 3. |
| `spec/parser.py` | Dialect fixes, `allOf` flattening, lower to the internal IR. |
| `spec/model.py` | The IR: `ParsedSpec`, `Operation`. |
| `jsonschema.py` | Emit JSON Schema 2020-12, `$defs`, inlining, body flattening. |
| `naming.py` | `operationId` → tool name (snake case, length cap, collision handling). |
| `filters.py` | Tag / path / method / operation filtering. |
| `toolset.py` | `Toolset`, `ToolSpec` — the compiled output. |
| `server.py` | `build_server`, `load_toolset`, `ServerApp`, hot-swap refresh loop. |
| `runtime/executor.py` | One tool call → one HTTP request. Owns the pool. |
| `runtime/auth.py` | api_key / bearer / basic / oauth2 / static headers. |
| `runtime/passthrough.py` | Per-caller credential forwarding, allowlisted. |
| `runtime/resilience.py` | Retries, circuit breaker, token bucket. |
| `runtime/response.py` | Render an HTTP response into MCP content, truncate. |
| `runtime/serializer.py` | `style` / `explode` matrix, body encoding. |
| `config.py` | The whole `Settings` tree. Precedence, env mapping, `${VAR}`. |
| `cli.py` | Typer app: `serve`, `deploy`, `inspect`, `call`, `generate`, `install`, `version`. |
| `install.py` | Write MCP client configs (Claude Code/Desktop, Cursor, project). |
| `codegen.py` | `generate` — emit a standalone deployable project. |
| `banner.py` | Terminal presentation. Owns the brand violet and the ASCII mark. |

## Invariants worth preserving

These are load-bearing. Changing one means changing tests and probably the README.

1. **Nothing downstream of `spec/parser.py` touches a raw OpenAPI dict.** Spec quirks are
   handled once, at the boundary. If you find yourself reaching into a dict in
   `runtime/`, the fix belongs in the parser.

2. **Credentials never reach the model.** Parameters a configured credential supplies are
   stripped from the tool schema and injected at request time
   (`schema.hide_auth_parameters`). Do not weaken this.

3. **POST/PATCH are not retried by default.** `retry.retry_non_idempotent` is opt-in.
   Re-sending a billable request is worse than failing.

4. **Passthrough is an allowlist, and `authorization` is deliberately not in it.** Adding
   it silently would forward bearer tokens no one intended to forward.

5. **stdout is sacred on stdio.** JSON-RPC framing lives there. Banners, logs, and
   diagnostics all go to stderr. Colour is suppressed when not a TTY, when `NO_COLOR` is
   set, or when `TERM=dumb`.

6. **Config models are `extra="forbid"`.** An unknown key is a startup error, not a
   silent no-op. Keep it that way — it is the main defense against typo'd config.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [The-Swarm-Corporation/mcp-scribe](https://github.com/The-Swarm-Corporation/mcp-scribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
