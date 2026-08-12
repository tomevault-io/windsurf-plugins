---
trigger: always_on
description: MCP server: AI read/write Polarion ALM. FastMCP 3.0, strict async, fully typed.
---

# CLAUDE.md

MCP server: AI read/write Polarion ALM. FastMCP 3.0, strict async, fully typed.

## Commands

```bash
uv sync --dev                                            # install deps
uv run pytest                                            # all tests
uv run ruff check . && uv run ruff format . && uv run mypy src/  # lint + format + types
uv run pytest --cov=src/mcp_server_polarion --cov=evals --cov-report=xml \
  && uv run diff-cover coverage.xml --compare-branch=origin/main --fail-under=90  # changed-line gate — run before push
uv run mcp-server-polarion                               # run server (stdio)
```

CI: same order + `ruff format --check` + `pytest --cov-fail-under=90`; diff-cover changed lines ≥90% cover `src/` + `evals/` both — incl parser defensive branches + `evals/harness` request handlers.

## Architecture

- `core/` — `client.py` (async httpx; serialize + pace + retry per Gotchas), `exceptions.py`, `config.py` (`POLARION_URL`/`POLARION_TOKEN`), `logging.py` (stderr-only).
- `tools/` — domain module per resource; `_build_*_payload` = unit-test seam; `tools/__init__.py` import registers `@mcp.tool`s. `_shared/`: `parse.py` (JSON:API→models), `pagination.py`, `fields.py`/`custom_fields.py` (sparse-fieldset + custom-field policy), `cache.py` (`TTLCache`), `sql.py`, `guard/` (pre-write validation, submodule per domain axis; new guards compose `_http.py` `guarded_get`/`guarded_pages` + `_custom_keys.py` `check_custom_keys`). `tools/guides/` = on-demand data served by `recipes.py`.
- `middleware.py` — compact tool-arg `ValidationError` to one-line summary (raw Pydantic dump = token waste).
- `utils/html.py` — Markdown↔HTML, `stamp_block_ids`, `first_anchorless_block`.
- `models/` — Pydantic v2, re-exported from `models/__init__.py`; `PaginatedResult[T]` wrap list responses.
- `server.py` — FastMCP instance; lifespan owns `PolarionClient`.

## Non-Negotiable Rules

- NEVER `print()` — stdout = MCP JSON-RPC; log to stderr.
- NEVER `typing.Any` — concrete types or `object`.
- All functions: full annotations + `from __future__ import annotations`. Tool functions: `async def` return Pydantic model.
- Body fields asymmetric by tool purpose:
  - Round-trip: `get_*(include_*_html=True)` return raw Polarion HTML; `update_*(*_html=...)` accept verbatim — no sanitize/convert.
  - Greenfield create (Markdown): `markdown_to_html` + `sanitize_html`. Post-create edits = raw-HTML round-trip; formats never mix.
  - Synthesis (READ-ONLY): `read_*` convert HTML→Markdown; feed output back to writes lose Polarion markup.
- Write payloads skip `None`/empty (Polarion read empty as "clear default"). Resource POSTs wrap in `{"data": [...]}`; action endpoints (`.../actions/<name>`) take flat object.
- Every list tool: `page_size` (max 100) + `page_number` → `PaginatedResult[T]` with `has_more`.
- Timestamps (where Polarion serve both): `list_*` summary = `updated` only; metadata-bearing `get_*`/`read_*` detail add `created` (read_document body-only synthesis — exempt). Domain without `get_*` tool: list expose what API serve (comments `created`-only). API without timestamps: omit.
- Every write tool: `dry_run: bool = False` — return payload, no hit Polarion.
- NEVER ship delete tool for unrecoverable data (attachment, work item, document) even where REST allow it (test record attachment DELETE = 204) — removal = human via portal. `dry_run` + `destructiveHint` = advisory, model still call with `dry_run=False`; withhold capability = only hard guarantee. Reversible relationship delete allowed (`delete_work_item_links` — recreate from context, zero data loss). Deliberate posture, not gap (#224 closed) — no re-litigate per review round.
- Error mapping: `PolarionNotFoundError`→`ValueError`, `PolarionAuthError`→`PermissionError`, `PolarionError`→`RuntimeError`; user-fixable status allowed narrower map (attachment 409 dup fileName→`ValueError`).
- Guards fail closed: validation GET error block write; only successful empty option set defer to Polarion.
- Docstrings = LLM manual, Google-style; only prose above `Args:` ship — keep tight; return-field bullets sync with model. Field descriptions one line, skip when name + type say all.
- Tool description template (order, skip empty):
  - [1] verb-first what + hard limits; [2] sibling routing ("— use X instead"); [3] call strategy only if behavior-change (REPLACE / "Call X first" / Atomic).
  - [4] round-trip format rules; [5] returns + follow-up; [6] errors as prevention-form ("resolve via list_*_enum_options first").
  - Shipped text = docstring prose + `Field(description=...)` + input spec-model class docstrings (`$defs` ship them) — NEVER exception class names / raw HTTP codes / RST double-backticks; caps only NEVER/REPLACE/Atomic; `dry_run` = approved byte-exact variants.
  - Budget: read ≤~50, write ≤~150 words. Gate `test_tool_description_style.py`; eval-FAIL-restored phrase = lock via docstring contract test.
- No `WARNING:`/`NOTE:` prefixes, dev-narrative, banner dividers. CLAUDE.md dev-only — MCP-user info live in `@mcp.tool` docstrings. Module docstring = why module exist; constraints inline next to what they constrain.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devemberx/mcp-server-polarion](https://github.com/devemberx/mcp-server-polarion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
