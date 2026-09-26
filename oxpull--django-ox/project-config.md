---
trigger: always_on
description: Exact steps for setting up django-ox in an existing Django project, the facts
---

# For AI assistants

Exact steps for setting up django-ox in an existing Django project, the facts
to get right while doing it, and how to prove it works. Machine-readable
copies: [llms.txt](llms.txt) (facts and links) and
[llms-full.txt](llms-full.txt) (every page of this site in one file).
Context7 library id: `/oxpull/django-ox`.

## Set up django-ox in this project

Requires Python 3.12+ and Django 5.2+. Check before installing:

```
python -c "import django, sys; print(django.__version__, sys.version.split()[0])"
```

Django 6.0 and later ship the Tasks framework in core. On Django 5.2 LTS it
comes from the `django-tasks` backport, so install the `backport` extra there.

Install:

```
pip install django-ox
```

or, with uv:

```
uv add django-ox
```

On Django 5.2 LTS:

```
pip install "django-ox[backport]"
```

or, with uv:

```
uv add "django-ox[backport]"
```

Edit `settings.py`:

```python
INSTALLED_APPS = [
    # ...
    "django_ox",
]

TASKS = {
    "default": {
        "BACKEND": "django_ox.backend.OxBackend",
    }
}
```

Create the table:

```
python manage.py migrate django_ox
```

Verify. The expected output is the second line:

```
python manage.py ox_health
OK: backlog=0 oldest_age=none last_claim_age=none
```

`manage.py check` also runs the django-ox system checks, so a bad schedule
or timeout option fails here, as `django_ox.E002` to `E005` and `E010`, before anything deploys.
Review any `django_ox.W003` warning before deploying too.

Start a worker in its own process, next to the web server, under the same
supervisor:

```
python manage.py ox_worker
```

Settings with every option named, for when the defaults need changing:

```python
TASKS = {
    "default": {
        "BACKEND": "django_ox.backend.OxBackend",
        "QUEUES": ["default"],  # [] allows any queue name
        "OPTIONS": {
            "MAX_ATTEMPTS": 3,  # claims per task before FAILED
            "LOCK_TIMEOUT": 300,  # seconds a worker may stop renewing its lease
            "BACKOFF_INITIAL": 5,  # first retry delay, seconds; doubles each attempt
            "BACKOFF_MAX": 600,  # retry delay ceiling, seconds
            "TASK_TIMEOUT": None,  # seconds one attempt may run; None is no limit
            "TASK_TIMEOUTS": {},  # per-queue values, {"queue": seconds}
            "TASK_TIMEOUT_GRACE": 30,  # seconds a timed-out thread gets to stop
            "SCHEDULES": {},  # recurring tasks, see Recurring tasks
        },
    }
}
```

## Facts to get right

- `QUEUES` sits beside `OPTIONS`, not inside it. Inside `OPTIONS` it is
  ignored without warning; the symptom is `InvalidTask: Queue 'X' is not
  valid for backend.`
- Tasks are plain Tasks-framework tasks. `from django.tasks import task` on
  Django 6.0+, `from django_tasks import task` on 5.2 LTS,
  decorate with `@task`, call `.enqueue(...)`. Nothing is imported from
  `django_ox` in task code.
- The worker imports a task by its dotted path, so the module must be
  importable in the worker process and the worker runs the same code as the
  producer; nothing is registered and there is no autodiscovery. `async def`
  tasks run.
- Priority and deferral are Django API: `task.using(priority=N)` with N from
  -100 to 100, higher first, and `task.using(run_after=...)` with a timedelta
  or datetime.
- Tasks run only while `ox_worker` is running. It is a separate process.
- SIGTERM and SIGINT both drain and exit 0; a second signal forces an
  immediate exit with code 130.
- `enqueue()` is one INSERT on the database the router sends `OxTask` to,
  `default` unless you wrote a router. Two things make the commit joint: a
  `transaction.atomic()` opened on that database, because a bare `atomic()`
  opens on `default`; and the caller's own rows written there too. With both,
  the task and those rows commit or roll back together, and the task is
  visible to workers only after commit. Do not add `transaction.on_commit()`
  around it. Rows written on another connection give two transactions, not
  one.
- Many calls of one task go through `django_ox.bulk.enqueue_many(task,
  [(args, kwargs), ...])`: one INSERT per 1,000 rows, one transaction, results
  in input order. Set queue, priority and `run_after` once with `.using(...)`.
- Execution is at-least-once. Write tasks to be safe to run twice: guard on
  state already in the database, not on a flag in memory.
- The task function runs outside any transaction. Open
  `transaction.atomic()` inside the task when it needs `select_for_update()`.
- An attempt is consumed at claim time, so a worker dying mid-run uses one.
  Retry delay after attempt n is `BACKOFF_INITIAL * 2 ** (n - 1)`, capped at
  `BACKOFF_MAX`.
- The worker schedules lease renewal every `LOCK_TIMEOUT / 3` seconds; task length is not bounded by `LOCK_TIMEOUT` while renewals succeed.
  Renewal needs a database connection: if the worker cannot refresh its lease for `LOCK_TIMEOUT`, the reaper can hand the task to another worker even while it is alive.
- With Django's PostgreSQL pool in 1.4.0, provide at least `concurrency + 1` pooled connections per worker process; add a spare pooled connection if fallback must work under full load.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oxpull/django-ox](https://github.com/oxpull/django-ox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
