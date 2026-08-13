---
trigger: always_on
description: MCP server exposing a semantic-search knowledge base over AT Protocol docs,
---

# AGENTS.md

MCP server exposing a semantic-search knowledge base over AT Protocol docs,
lexicon schemas, Bluesky API docs, and cookbook examples. Python 3.12+,
FastMCP-style `MCPServer` (`mcp` SDK >=2.0) + txtai hybrid (BM25 + dense)
embeddings, stdio transport.

## Commands

```bash
uv sync                                            # install (uv-managed, hatchling build)
uv run atproto-mcp                                 # run server over stdio
uv run mcp dev src/atproto_mcp/server.py           # MCP Inspector
uv run python -m unittest discover -s tests -p 'test*.py'  # test suite (what CI runs)
uv run python tests/smoke_test.py                  # manual end-to-end check (see gotcha below)
```

- No linter, formatter, or type-checker is configured. Match existing style by hand.
- CI (`.github/workflows/tests.yml`) installs with plain `pip install .` on
  Python 3.12 and runs the `unittest` discover command above.
- `pyproject.toml` requires Python >=3.12 (matches README's "Python 3.12+"
  prerequisite).

## Architecture

Startup flow (`server.py`): `app_lifespan` → `Config.from_env()` → publish
`state.config` → `asyncio.create_task(_warmup(config))` → **yield
immediately**. Nothing slow runs on the startup path: a cold start clones four
repos and downloads a ~130MB model, far longer than a desktop host will wait
during the stdio handshake.

`_warmup()` then, off the startup path:
1. `KnowledgeBase.load()` via `asyncio.to_thread` — **before any network
   access**, so a warm start serves in seconds even offline. On success it
   publishes with `state.commit_kb(kb, generation, save=False)` (already on
   disk, do not rewrite) and sets status `ready`.
2. `fetch_all()` to clone/refresh repos.
3. Cold start: `parse_all()` + `kb.build(save=False)` + `commit_kb()`.
   Warm start: if the loaded index's repo SHAs (`kb.indexed_shas`) or
   embedding pipeline (`kb.indexed_pipeline`) differ from the current config,
   the live index keeps serving while `_rebuild_index()` builds a fresh
   `KnowledgeBase` via `asyncio.to_thread` and commits it.

`_warmup` never raises: exceptions are logged and recorded as status `failed`,
so the server stays connected and can explain itself. It then **retries with
exponential backoff** (`_RETRY_INITIAL_DELAY` → `_RETRY_MAX_DELAY`, both
module-level so tests can zero them) for as long as `state.kb is None` — the
common failure is a desktop host launching the server before the network is
up. Retrying stops once an index is serving; `refresh_sources` remains the
explicit recovery path and resets the status to `ready`.

`tests/test_warmup.py`'s `_StateIsolationMixin` zeroes both delays for every
test in the file — without that, any test that fails a cold start spins in the
retry loop and stalls the suite.

**Warmup status** (`state.py`): `set_status()` / `get_status()` /
`status_message()` over the `STATUS_*` constants
(`starting|fetching|indexing|ready|failed`). `get_kb()` raises
`KnowledgeBaseNotReady` (a `RuntimeError` subclass) carrying
`status_message()` when no index is in service. `tools.py` and `resources.py`
each define a local `_reports_warmup` decorator, applied under `@mcp.tool()` /
`@mcp.resource()`, that turns that exception into a normal response — plain
text for tools, a `{"status", "message"}` JSON object for resources. It uses
`functools.wraps` because the generated MCP schema reads the docstring and
signature; do not replace it with a plain wrapper. Progress statuses are set
only when **no** index is serving — a background refresh behind a live index
must stay `ready`.

**Rebuild concurrency** — two separate mechanisms, do not confuse them:

1. `state.index_build_lock` serializes the *work*. `fetch_all()` rewrites the
   repo clones, `parse_all()` reads them, and `KnowledgeBase.save()` rewrites
   `<cache>/index/` in place, so two overlapping builds would corrupt each
   other. Because `parse_all` swallows per-source exceptions, the damage is
   silent: a source whose tree was deleted mid-parse yields zero chunks, and
   the truncated index is saved stamped with *fresh* SHAs, which nothing ever
   invalidates. **Any new code path that fetches, parses, or saves must take
   this lock.** It is a plain `threading.Lock` — not reentrant, so never call
   `_rebuild_index` while holding it (`_warm_index` returns a "needs rebuild"
   flag instead of calling it directly). `refresh_sources` acquires it
   `blocking=False` and declines with a message rather than hanging the tool
   call for minutes.
2. The **generation token** guards *publication* only. Every rebuild claims
   one via `state.claim_generation()` before starting; `state.commit_kb(kb,
   generation)` saves to disk and swaps `state.kb` inside one locked critical
   section, discarding results whose generation is older than the one already
   published — a slow rebuild can never overwrite a newer index in memory or
   on disk. Always check the return value: `False` means your result was
   thrown away, so do not then log success or set status `ready`.

| Module | Role |
| --- | --- |
| `config.py` | `Config` (frozen dataclass, env-driven) + `REPOS` source registry + `SOURCE_*` constants |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ashex/atproto-mcp](https://github.com/Ashex/atproto-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
