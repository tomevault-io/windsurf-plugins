---
trigger: always_on
description: An open-source GPU cluster manager for running AI models. The Python package
---

# GPUStack

An open-source GPU cluster manager for running AI models. The Python package
is a single distribution that ships three runtimes — **server** (API + scheduler +
controllers), **worker** (runs inference backends on a node), and **CLI** — selected by
`gpustack <subcommand>`.

## Project

Stack: Python, FastAPI, SQLModel over SQLAlchemy (asyncio), Alembic, `uv` for dependency
management. Higress fronts inference traffic as the AI gateway. The shipped artifact is
a container image; the wheel is a build step inside it. Supported datastores are
PostgreSQL, openGauss, MySQL and OceanBase.

Directories whose contents are not obvious from the name:

```
gpustack/
  api/            auth, multi-tenancy (TenantContext), HTTP exceptions, middlewares
  client/         API clients; generated_*.py come from codegen/, never hand-edit them
  codegen/        Jinja templates that emit gpustack/client/generated_*.py
  config/         Config settings model — user-facing CLI flags / env / config file
  envs/           GPUSTACK_* operational tuning knobs, one module
  gateway/        Higress AI gateway integration, ext_auth, wasm plugins
  gpu_instances/  GPU Service: cloud GPU instances, their CRDs and controllers
  mixins/         ActiveRecordMixin, timestamp mixin
  policies/       scheduler worker_filters / scorers / candidate_selectors
  routes/         FastAPI routers; routes.py wires every router and its auth tier
  schemas/        SQLModel tables + Pydantic request/response models
  server/         server runtime: controllers, event bus, services, collectors
  worker/         worker runtime, inference backends, benchmark and cache managers
hack/             the scripts behind every make target
```

## Code style

Formatting and linting are enforced by pre-commit; run `make lint` before you finish.

- **black**, line length 88, `skip-string-normalization = true`. Do not convert `'…'` to
  `"…"` (or back) as a side effect — leave existing quoting alone.
- **flake8** (`.flake8`): `select = C,E,F,W,B,B950`, `ignore = E203,E501,W503,E701,E704`
  and pre-commit runs it with `--max-complexity=15`; that is the gate you must pass. A
  genuinely irreducible function gets `# noqa: C901` with the reason clear from context.
- `gpustack/migrations/` and `*/generated*` are excluded from black and flake8. Match
  the surrounding file's style there anyway.
- **shellcheck** runs over `hack/*.sh` with `-x`. New scripts start with
  `set -o errexit -o nounset -o pipefail` and source `hack/lib/init.sh`, like the
  existing ones.
- **Logging**: `logger = logging.getLogger(__name__)` at module top. There is a custom
  `TRACE` level (`gpustack/logging.py`) available as `logger.trace(...)` at runtime.
- Type hints on public functions, `Optional[X]` / `List[X]` from `typing`; docstrings in
  the Google-ish `Args:` / `Returns:` form seen throughout `gpustack/utils/`.
- **Comments explain what the code does and why — never the change history.**
  Never write comments like "previously X, now Y", "renamed from…", or references to a
  commit, PR, or past bug. The same applies to test comments: state the invariant being
  verified, not the bug that motivated it.

## Security-sensitive changes

A mistake in these leaks data rather than breaking a build, so review them deliberately
and say in the PR what you checked:

- **A new or changed route** — pick the tier deliberately in `gpustack/routes/routes.py`
  (any authenticated user / tenant-scoped / platform admin / worker system principal).
- **Any query a tenant can reach** — filter by `TenantContext`: `tenant_list_conditions`
  for lists, `assert_resource_visible` and friends for single rows. Unfiltered means
  other Orgs' rows come back. When in doubt, deny.
- **Auth, API keys, tokens, SAML, cloud credentials** — never log a secret, never echo
  one back in a response.
- **A migration touching an ownership or permission column** — it silently rewrites who
  can see what.

## Database migrations

Migrations are Alembic revisions under `gpustack/migrations/versions/`.

Rules:

- **Released revisions are immutable.** Changing one breaks upgrades for anyone already
  on it.
- **Unreleased schema changes get folded into the current release's bundle revision**
  (e.g. `…_v2_3_0_database_changes.py`) rather than accumulating one file per PR. Add a
  standalone revision only when the change must be separately ordered; expect it to be
  folded in before the release.
- **Every migration must work on PostgreSQL, openGauss, MySQL and OceanBase.** openGauss
  connects as the `postgresql` dialect and OceanBase as `mysql`, so `dialect.name`
  reports only two values for the four databases. Detect the compatible ones explicitly,
  the way `is_opengauss` in `gpustack/migrations/utils.py` does. The differences are
  real: enum types are shared in PostgreSQL but inlined per column in MySQL (see
  `gpustack/utils/sql_enum.py`), and openGauss lacks `jsonb_agg` /
  `jsonb_array_elements`.
- Make migrations re-runnable: guard with `table_exists` / `column_exists` from
  `gpustack/migrations/utils.py` rather than assuming a clean starting state.
- Write a real `downgrade()`, or leave `pass` with a comment explaining why the change
  is not reversible.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gpustack/gpustack](https://github.com/gpustack/gpustack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
