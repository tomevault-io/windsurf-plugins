---
trigger: always_on
description: validates the whole body and **writes nothing at all** when it refuses.
---

# src/application/ — the application

Python 3, Flask + APScheduler + DuckDB, served by uvicorn. The commands are in
the root `CLAUDE.md`; the *why* of each choice is in `docs/adr/`, then in
`docs/v5-decisions.md`.

## The boot

`src/application/boot.py` is the container `ENTRYPOINT` **and** the boot sequence, and since
ADR-0039 it is linear — the app does not fork:

1. **the environment** — `boot_env.read()`, once and as a whole.
2. **`main.build_runtime()`** — the store (opened, DDL applied, seeded, and
   **kept open** for the life of the process), `ConfigurationManager`, the first
   publication of the config snapshot.
3. **`api.create_app(runtime)`** — the Flask app, on that runtime. It no longer
   builds one of its own: the factory used to be named on gunicorn's command
   line, which is why it could.
4. **`main.start_runtime()`** — the `BackgroundScheduler` and the first
   `ingest()`. It opens nothing.
5. **`boot.serve()`** — `uvicorn.run()` in process, on `Serving`: the Flask app
   behind a WSGI→ASGI adapter, no route rewritten.

A failure at any of the five is **one** non-zero exit (`boot.run`). There used to
be two answers — the master exited 1, `post_fork` re-raised so gunicorn would
halt the arbiter rather than respawn a worker that would fail identically — and
there is no arbiter.

**The teardown is the lifespan shutdown**, not a `finally` after `uvicorn.run`:
uvicorn catches `SIGTERM`, shuts down gracefully, then re-raises the signal, so
the process dies the instant `run()` returns. `main.shutdown_runtime` is called
from `Serving`'s `lifespan.shutdown` — the heir of `worker_exit` — and again from
the `finally`, which covers a boot that never bound a socket at all. It is
written to be called twice.

**One process is structural, not guarded.** `on_starting` and
`control_socket_disable` were two guards spent forbidding a second worker — N
workers would be N schedulers — and they left with the arbiter. uvicorn has a
multiprocess mode, and the door stays shut by the image having **no server
command line**: the `ENTRYPOINT` is `python boot.py`. Concurrency comes from the
WSGI thread pool (`boot.WSGI_THREADS`). **One socket, one application**
(ADR-0033): `SB_WEB_PORT` and nothing else, and the whole app — the page, `/api`
writes included, `/health` — answers on it. The second socket the exporter used
to be bound to is gone, and with it the "publish only the metrics port" that hid
nothing anyway: whoever reaches the socket reaches the whole app.

**`/health` says health in two registers that never mix** (ADR-0036). The
**status code** is the orchestrator's, and its predicate is the one #696 settled
— the worker serves and the store answers; nothing else reaches it. The **body**
is a person's: each job (scrape, backfill, performance) with its last pass and
its verdict, plus one word for the whole. A job that is late, wedged or silent
is amber **with a `200`** — restarting repairs nothing yfinance broke, and a
probe that reddened on it would turn a stuck job into a restart loop. The body
is folded by `runtime_view.build_health` out of the recorder's last-pass records
and issues no query, which is what keeps the two registers apart; when the store
goes, so does the body, and the `503` is the whole answer. The route keeps its
name: `/healthz` was examined and declined.

## The store

`store.py` owns the file: the connection, the DDL of the twelve tables, the seed.

- **One thread inside the connection at a time**, reentrant lock;
  `Store.transaction()` holds it from `BEGIN` to `COMMIT` (a transaction on one
  connection is visible to every thread using it).
- **DDL with `IF NOT EXISTS`, no migration machinery.** A new column would exist
  on no store created before it: derive at read time instead.
- **Every table has exactly one writer** — the configuration path owns
  `account`/`symbol`/`installation_fact`, `advisories.py` owns `advisory_ack`
  (one write path, and it is the acknowledgement), `entries.py` owns `event`
  **whole**
  (ADR-0032, #816: one population, one writer, the named exception of
  `reassignment.py` apart), the ingestion `position`/`account_state`, the market
  `symbol_quote`/`price_point`, the perf job
  `account_metrics`/`portfolio_totals`.
- **`setting` is the one table with two named writers**: the configuration path
  answers for a human's choice, and `entries.py` upserts two keys of its own —
  the reporting currency a file declares, and `ledger_last_write` (`_stamp_write`,
  inside the gesture's own transaction, since #816 left no `import_source` row to
  read the instant off).
- **`price_point` carries no primary key and no foreign key** (ADR-0007): a DuckDB
  ART index is a second copy of the data in resident memory (+563 MB on a 319 MB
  base). Uniqueness moves to the writers.
- **A NaN is not a number** (`store.finite`), neither stored nor served — JSON has
  no NaN.
- **Two kinds of time, never mixed**: `TIMESTAMPTZ` in UTC for an observed
  instant, `DATE` for a calendar day. A bound on a `DATE` column is **cast**, or
  DuckDB widens it to midnight and the first day of every window is dropped.
- **The seed has two halves**: the `default` account row is written at creation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pbrissaud/suivi-bourse](https://github.com/pbrissaud/suivi-bourse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-10 -->
