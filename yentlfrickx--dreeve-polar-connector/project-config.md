---
trigger: always_on
description: <!-- Persistent memory across AI sessions. Agents read this at pipeline start
---

# Compound Learning

<!-- Persistent memory across AI sessions. Agents read this at pipeline start
     and update it when they discover new patterns, gotchas, or decisions.
     Review new entries with the same scepticism you'd apply to generated content.
     Only record observed reality — not aspirations. -->

## STYLE

- Atomic file writes: always write to `{path}.fit.tmp` then `os.rename()` to final path; DB record inserted only after rename. Pattern used in `sync.py:_atomic_write`.
- APScheduler async jobs: register `async def` callables with `AsyncIOScheduler` directly — it awaits them and surfaces exceptions. Never wrap in a sync function calling `ensure_future` (swallows exceptions into orphan tasks).
- OAuth manual implementation: use raw `httpx` with `Authorization: Basic base64(client_id:client_secret)` header and form-body POST. No `authlib` needed for a single authorization-code exchange.
- FIT-byte parsing is isolated in one function (`sync.py:_parse_fit_sport`) so `fitparse` usage stays in one place, mirroring the existing convention of isolating third-party/IO concerns (`polar.py` for HTTP, `db.py` for SQL).
- Prefer recomputing a pure decision function (e.g. `_passes_filter(stored_value, current_config)`) over persisting a config signature/hash for cache-invalidation. A skip-tracking feature (`skipped_exercise` table, see ARCH_DECISIONS) originally proposed a `filter_signature` column + comparison scheme; an adversarial review (`OBJECTIONS_SKIP_TRACKING.md` O1) found it unnecessary because the underlying pure filter function can just be re-run against the stored value at read time — same correctness, one fewer column, no signature-staleness bugs. Reach for this pattern whenever "did config X change in a way that invalidates cached decision Y" comes up and Y was computed by a pure function.
- A resource's parent directory should be created by the module that owns/opens that resource, not by every caller. `Db.__init__` (`db.py`) now creates `pathlib.Path(path).parent` (guarding `":memory:"` and bare-filename/no-dir-component paths) instead of relying on callers to `mkdir` it first. This covers every entry point (`web.py`'s `create_app`, `__main__.py`'s `sync` command, and all tests) with one edit, and is why `PFS_DB_PATH` and `PFS_OUTPUT_DIR` (which still gets its own `mkdir` in `__main__.py`/`sync.py`, since no single module owns "the output dir") can safely point at totally unrelated directory trees.
- FastAPI route query params meant to tolerate arbitrary/malformed user input (pagination/sort controls) should be typed `str` with manual parsing/clamping in the handler body, not `int`/`Literal` directly in the signature — Pydantic auto-rejects malformed values with a 422 before any normalization code runs, when the requirement is "always render 200 with a graceful fallback." Used in `web.py`'s `GET /activities` route (`page`, `page_size`, `sort`, `dir` all `str`).
- For a paginated/sortable/filterable list query (`Db.list_downloaded()` in `db.py`), allow-list sortable columns via a fixed module-level dict (`LIST_SORT_COLUMNS`) and validate direction against a literal `("asc","desc")` tuple — never interpolate a user-supplied column/direction string into `ORDER BY` directly. Always append a deterministic tiebreaker column (here, `exercise_id ASC`) so `LIMIT`/`OFFSET` pagination returns stable, non-overlapping pages across requests when the primary sort column has ties.

## GOTCHAS

- `Dockerfile` editable install: `pip install -e .` with hatchling requires `src/` to be present first. Either use `pip install .` (non-editable) or `COPY src/ src/` before the pip install step. The original Dockerfile had this wrong and was fixed by the deploy agent.
- SQLite + NFS: never put a SQLite DB on an NFS-backed RWX PVC with multiple writers. The architecture was redesigned to use a single Deployment (one pod, one writer) with an RWO PVC to avoid this.
- `AsyncIOScheduler` + sync wrapper: a synchronous job calling `asyncio.get_event_loop()` is deprecated in Python 3.12+ and loses exceptions. Always register async job functions.
- Polar `expires_in`: the token exchange response includes `expires_in` even though long-lived. Store it and detect expiry by computation + 401 responses. Never assume tokens are permanent.
- `with_suffix(".tmp")` replaces the extension; `with_name(name + ".tmp")` appends. Use the latter for temp files so the original extension is preserved.
- Polar AccessLink's exercise-list `sport` field is coarser than the FIT file's own session `sport`: distinct activity types (e.g. walking vs. generic/uncategorized) can both report as `OTHER` at the API level. Never trust `ex.sport` for fine-grained classification — parse the downloaded FIT bytes (`fitparse`, first `session` message) when precision matters, and treat the API field as a fallback only.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YentlFrickx/dreeve-polar-connector](https://github.com/YentlFrickx/dreeve-polar-connector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
