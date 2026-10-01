---
trigger: always_on
description: How to WRITE tests here. For how to RUN them (suite invocations, compose services, the
---

# django-absurd — test-authoring conventions

How to WRITE tests here. For how to RUN them (suite invocations, compose services, the
pre-commit gates), see [`../CLAUDE.md`](../CLAUDE.md).

- pytest, **function-based only** (never class-based).
- **Non-fixture test helpers live in a `utils.py`** module (never `support.py` or other
  invented names) — e.g. `tests/utils.py`, `tests/core/test_admin/utils.py`,
  `tests/pg_cron/utils.py`. Import the module (`from tests import utils`) and qualify.
- **Same for the fixture task modules**: `from tests import tasks` /
  `from tests import atasks`, then `tasks.add`, `tasks.routed`, `atasks.aecho`. Never
  `from tests.tasks import routed` — a bare adjective at the call site says nothing
  about what runs, and it forces rename-aliases like `make_group as make_group_task`.
- **`from pytest_django import Settings`**, always, and annotate bare:
  `settings: Settings`. Never `pytest_django.fixtures.Settings`, quoted or otherwise — a
  quoted one passes the test run and fails mypy, so it survives until the slow gate.
- **Assert a boolean by identity: `is True` / `is False`**, never `assert x` or
  `assert not x`. Applies to every `.exists()` — `assert qs.exists() is False`, not
  `assert not qs.exists()`. Truthiness passes for the wrong object too: drop the
  `.exists()` call in a refactor and `assert qs` still passes on a non-empty queryset,
  while `is True` fails. Same for any predicate helper returning `bool`.
- **Shared fixtures live in the parent `tests/conftest.py`**, inherited by all four
  suites via `--confcutdir=..` in each suite's `pytest.toml` (each suite's rootdir is
  its own dir, so without `confcutdir` a parent conftest isn't discovered). Do NOT
  re-import fixtures into a suite conftest — a suite `conftest.py` holds only
  suite-specific fixtures. Per-test pg_cron isolation is not a suite-local fixture; it
  comes from the mechanisms described in [`../CLAUDE.md`](../CLAUDE.md).
- An **autouse `_enable_db(db)` fixture** (in `tests/conftest.py`) gives every test DB
  access — do NOT decorate tests with `@pytest.mark.django_db`. Only add
  `@pytest.mark.django_db(transaction=True)` (or markers for multi-DB / reset-sequences)
  when a test needs transactions/commits or DDL (`migrate`, `create_queue`).
- **Any test that EXECUTES anything — enqueue, drain, a worker, cleanup deleting rows —
  freezes time through the `dj_absurd` fixture**, not through time-machine directly:
  `with dj_absurd.freeze_time() as frozen_time:`, then
  `frozen_time.shift(Δ)`/`move_to(instant)`, enqueueing INSIDE the block — **never
  `time.sleep`**. It moves Postgres and Python together, which is mandatory: Postgres
  ahead of Python is an unkillable deadlock for a sync task. The fixture works unchanged
  in an `async def` test. See
  [Testing — the `dj_absurd` fixture](../docs/web/testing.md#the-dj_absurd-fixture).
  - **`tests/benchmarks` is exempt.** It drives a measurement harness through real
    `absurd_worker` children and real sleeps, and elapsed time on a real clock is the
    thing being measured; freezing either clock erases it.
- **`time_machine.travel(..., tick=False)` directly is for pure-Python math only** —
  cron arithmetic (`get_next_datetime`) and the like, where no row, worker, or Absurd
  deadline is involved. Reaching for the fixture there would write a database GUC for
  nothing; reaching for time-machine on an executing test leaves Postgres on real time
  (that mistake shipped once — `test_cleanup.py` passed only because `cleanup_ttl` was
  0). Two ticking uses are sanctioned:
  - `tests/core/test_scheduler.py`'s live worker crossing a `*/1` boundary, which needs
    real time to pass.
  - `tests/benchmarks/utils.py`'s `nap_the_wall_clock`, which walks `time.time` away
    from `perf_counter` on purpose — that disagreement is the only input the harness's
    suspension guard reads, so here the drift IS the phenomenon under test. The
    `dj_absurd` fixture is wrong twice over: it moves Postgres too, erasing the
    disagreement, and its GUC never reaches an `absurd_worker` child. Safe only because
    nothing in the harness's own process derives a database deadline from the wall clock
    — every drain deadline is monotonic and every recorded timestamp is a Postgres
    column.
- **freezegun is banned** — it patches `time.monotonic`, which IS asyncio's event-loop
  clock, so a frozen freezegun deadlocks the drain unkillably. Do not reintroduce it.
  `pytest-asyncio` is a dev dependency for writing `async def` tests; nothing in
  `django_absurd/` may depend on it.
- **A test needing the Absurd schema out of reach uses `utils.hide_absurd_schema()`** —
  it renames the schema and renames it back, so nothing is destroyed and no migration
  state moves. Never unapply migrations for this: it replays the whole install per test,
  and it stops working outright once a schema delta lands (Absurd publishes no downgrade
  SQL).
- **No monkeypatching / `unittest.mock.patch`.** Test observable behavior, not
  internals. If a test needs to patch our own functions to reach a branch, restructure
  so a real input drives that branch instead.
  - **One carve-out: the resolver that names the central pg_cron database.** An

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lincolnloop/django-absurd](https://github.com/lincolnloop/django-absurd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
