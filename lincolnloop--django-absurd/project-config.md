---
trigger: always_on
description: Django app wrapping [Absurd](https://earendil-works.github.io/absurd/) (Postgres-native
---

# django-absurd — project instructions

Django app wrapping [Absurd](https://earendil-works.github.io/absurd/) (Postgres-native
workflow engine). Package at repo root (`django_absurd/`, no `src/`). Specs live in
`docs/specs/`, plans in `docs/plans/`.

This file is about **maintaining** the project — conventions, testing, tooling. For
how-to / integration / usage (configuring the backend, enqueuing, workers, releasing),
see [`django_absurd/AGENTS.md`](django_absurd/AGENTS.md), the user-facing guide; don't
duplicate that material here.

## Naming

- **Functions must contain a verb** (`get_declared_queues`, `sync_queues`,
  `check_absurd_queues`) — never a bare noun (`queue_policies`, `absurd_client`). Avoid
  pointless `_`-prefixed helpers; if a helper exists, give it a real verb-name.
- Exception: autouse pytest fixtures never called directly (e.g. `_enable_db`) may keep
  the `_` + plain-name form.
- **Test fixture tasks read at their call site, not their definition.** The shared ones
  in `tests/tasks.py` / `tests/atasks.py` are always reached module-qualified
  (`tasks.capped`, `tasks.routed` — see Testing conventions), so a terse adjective name
  is fine there: the module supplies the missing noun. A task defined **locally in a
  test module** has no such prefix, so it must carry the verb itself
  (`make_group_on_immediate_backend`, `echo_int`), never a bare property or provenance
  (`off_backend`, `defined_elsewhere`, `plain`, `folded`). When a test binds a resulting
  `Task` _object_ to a local name, prefix it `task_` (`task_with_folded_defaults`) — the
  object is a noun, the function is not.
- **No leading-underscore module constants or helpers** — use plain names
  (`MUTABLE_OPTION_KEYS`, not `_MUTABLE_OPTION_KEYS`).
- **Module layout:** put helper functions BELOW the public function(s) that use them.

## Imports

- **Always `import typing as t`** — never `from typing import X`. Use `t.Any`,
  `t.TYPE_CHECKING`, `t.Sequence`, etc.
- **Absolute imports only** — no relative imports. Enforced by ruff
  (`ban-relative-imports = "all"`).

## Comments

- **A comment answers "why this, not the obvious alternative" — in ≤2 lines.** Longer
  reasoning goes in the commit message (why we changed it), `docs/WHY.md` (why the
  design is this shape), or a spec.
- **Delete-test:** if removing it costs a reader nothing the code already tells them,
  delete it. Never restate what the code does, narrate rejected alternatives, or
  describe what the code used to be.
- **Exception: write it out when the reason lives outside the code.**
  `names_a_queue_table` in `queues.py` explains that Postgres populates no
  `diag.table_name` for that error, which is why the match reads `message_primary`.
  Nothing in the code says that, so deleting the comment invites the next edit to undo
  it.

## Django system-check messages

- `msg` states the PROBLEM only; `hint` states the RESOLUTION. Never duplicate fix text
  in both.

## Exception hierarchy

- django-absurd raises its **own** exception types for its own failure modes, all under
  `DjangoAbsurdError`, defined in `django_absurd/exceptions.py`. Prefer a specific type
  over a bare stdlib/Django one when the condition is specific to this package.
- The type name carries the condition (`QueueNotDeclaredError`,
  `QueueNotProvisionedError`), and **the exception owns its message** — constructors
  take the data, callers never assemble text and no `format_*` helper is imported to
  build one.
- Named for the distributing package, not the upstream SDK: `DjangoAbsurdError`, never
  `AbsurdError`, because modules import from both `absurd_sdk` and `django_absurd` and
  the short name reads as the SDK's.
- Be honest about coverage: `except DjangoAbsurdError` catches the typed errors, not
  every error the package can raise — plain `ImproperlyConfigured`/`RuntimeError`/
  `TypeError` remain in `checks.py`, `connection.py`, and `test.py`'s guards for now.

## Exception chaining

- Re-raising a curated error inside an `except` always chains with `from exc` — never
  `from None`. Add `as exc` to the handler if it doesn't already bind a name.
  `from None` hides the real cause exactly when the curated message turns out to be the
  wrong guess.
- Pair this with narrowing the catch: classify first, re-raise the original untouched
  when the error isn't about what your curated message claims, chain with `from exc`
  when it is. `from exc` is not a licence to relabel broadly — see `names_a_queue_table`
  in `django_absurd/queues.py` for the worked example of both together.

## Testing

Test-authoring conventions live in [`tests/CLAUDE.md`](tests/CLAUDE.md) — read it before
writing or editing any test file. Running the suites:

- Tests run on the HOST via uv/tox (no app container). Three suites, each with its own
  `pytest.toml` and settings; invoke explicitly (a bare `uv run pytest` at repo root
  collects nothing and exits code 5 — intentional):
  - `uv run pytest tests/core` — core django-absurd; `django_absurd.pg_cron` NOT
    installed; plain `db` service (`PGPORT`, default 5432).
  - `uv run pytest tests/pg_cron` — pg_cron app installed; requires the `db_pg_cron`
    service (`PGPORT_PGCRON`, default 5434); an ORDINARY test DB (`test_absurd_pg_cron`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lincolnloop/django-absurd](https://github.com/lincolnloop/django-absurd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
