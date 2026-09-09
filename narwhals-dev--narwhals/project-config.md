---
trigger: always_on
description: Narwhals is an extremely lightweight, zero-dependency compatibility layer between Python dataframe libraries.
---

# AGENTS.md

Narwhals is an extremely lightweight, zero-dependency compatibility layer between Python dataframe libraries.

It lets library authors write dataframe-agnostic code once using a subset of the Polars API,
and have it work across pandas, Polars, PyArrow, cuDF, Modin, Dask, DuckDB, PySpark, Ibis, and SQLFrame,
without depending on any of them.

The primary audience is **library maintainers**, not end users.
Because of that, stability and backwards compatibility are taken extremely seriously.

## Read the docs first

Almost everything an agent needs is already documented. Read the relevant page instead of inferring
from the source, and update the page when you change the behaviour it describes.

| Topic | Read |
| --- | --- |
| Internal architecture: expressions, nodes, expression metadata, broadcasting, `over` push-down, group-by | [docs/how_it_works.md](docs/how_it_works.md) |
| Contributor workflow: env setup, test invocations, backend-specific rules, docstring style, PR conventions | [CONTRIBUTING.md](CONTRIBUTING.md) |
| Stable API guarantees and the `main` vs `stable.v1` / `stable.v2` diff | [docs/backcompat.md](docs/backcompat.md) |
| Adding a backend: compliant protocols, plugin entry points, the IO namespace contract | [docs/extending.md](docs/extending.md) |
| Row-order semantics: what `DataFrame` guarantees, what `LazyFrame` doesn't, `over(order_by=...)` | [docs/concepts/order_dependence.md](docs/concepts/order_dependence.md) |
| Null vs NaN: which methods exist for which, and what pandas muddies | [docs/concepts/null_handling.md](docs/concepts/null_handling.md) |
| Why the pandas `group_by` `UserWarning` exists and how to avoid triggering it | [docs/concepts/improve_group_by_operation.md](docs/concepts/improve_group_by_operation.md) |
| Boolean semantics, the pandas index, non-string column names | [docs/concepts/](docs/concepts/) |
| Which methods each backend implements | [docs/api-completeness/](docs/api-completeness/) (generated, do not hand-edit) |
| Public API surface | [docs/api-reference/](docs/api-reference/) (member lists are validated by CI) |
| `narwhals.sql`: generating SQL from Narwhals expressions | [docs/generating_sql.md](docs/generating_sql.md) |
| Security reporting and release-permission policy | [docs/security.md](docs/security.md) |

The one-sentence summary of [docs/how_it_works.md](docs/how_it_works.md), worth internalising before
touching anything in `_pandas_like/`, `_arrow/`, or `_compliant/`:

> An expression is a function from a DataFrame to a sequence of Series.

## Code structure and design

Narwhals follows a layered design:

1. **Public API** ([src/narwhals/dataframe.py](src/narwhals/dataframe.py),
   [series.py](src/narwhals/series.py), [expr.py](src/narwhals/expr.py), ...): the user-facing
   Polars-like API. Thin wrappers that build `ExprNode`s and dispatch to compliant backends.
2. **Compliant wrappers**: one folder per backend (`_pandas_like/`, `_arrow/`, `_polars/`, `_duckdb/`,
   `_spark_like/`, `_dask/`, `_ibis/`), each backend implements Narwhals-compliant DataFrames,
   Series, Exprs, and Namespaces that translate the Polars-like API to native calls. Shared
   protocols and base classes live in `src/narwhals/_compliant/`.
3. **Native libraries** (pandas, Polars, PyArrow, ...): the actual computation engines, never
   directly depended on.

Outside `src/`, the project has:

- [tests/](tests/): test suite, see [CONTRIBUTING.md → Running tests](CONTRIBUTING.md#7-running-tests)
- [tpch/](tpch/): TPC-H benchmark queries, see [tpch/README.md](tpch/README.md)
- `packages/`: uv workspace members, built and installed independently of `src/narwhals`. Currently just a `test-plugin` that is a minimal fake backend used to test `narwhals.plugins`.

## Hard rules

These are non-negotiable and the most common source of review comments. The long form, with
rationale, is in [CONTRIBUTING.md](CONTRIBUTING.md).

* **Zero dependencies.** Narwhals must never add a runtime dependency. It only uses what the user
  passes in.
* **Never import anything for `isinstance` checks.** Use the functions in
  [src/narwhals/dependencies.py](src/narwhals/dependencies.py) (e.g. `is_pandas_dataframe`).
* **Never iterate over rows.** Assume infinite rows. Column iteration is acceptable.
* **Never modify user input data.** Especially with pandas: no inplace operations on user-provided
  objects.
* **100% branch coverage** is enforced by the full-coverage CI job. When a branch is genuinely
  unreachable (e.g. gated on an unsupported backend version), mark it `# pragma: no cover` with a
  one-line reason.
* **Breaking changes never land in `narwhals.stable.v1` or `narwhals.stable.v2`.** New public APIs
  land in the main `narwhals` namespace and graduate into the next stable version. See
  [docs/backcompat.md](docs/backcompat.md), and add an entry to its `main` vs `stable.*` diff when
  the namespaces diverge.

Backend-specific rules (no pandas `apply`/`map`/`assign`/`drop`/`reset_index`/`rename`, no Polars
`map_elements`, no ordering assumptions or materialisation on lazy backends, DuckDB Python API over
SQL) are listed in full under

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [narwhals-dev/narwhals](https://github.com/narwhals-dev/narwhals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
