---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A PostgreSQL extension (`pgbson`) implementing a `bson` data type backed by the MongoDB C driver's
`libbson`. Very little is tracked in git — the whole extension is one C file plus one SQL script:

| File | Role |
| --- | --- |
| `pgbson.c` | The entire C implementation (~1000 lines, no headers of its own) |
| `pgbson--2.1.sql` | Extension install script (current): type, casts, operators, opclasses, `CREATE FUNCTION` bindings |
| `pgbson--2.0.sql` | The previous install script, frozen. Kept shippable so `CREATE EXTENSION pgbson VERSION '2.0'` still works and the upgrade path stays testable |
| `pgbson--2.0--2.1.sql` | Upgrade delta: the one `CREATE FUNCTION bson_get_value` |
| `pgbson.control` | Extension metadata for `CREATE EXTENSION` |
| `Makefile` | PGXS build; top half is hand-edited local paths |
| `pgbson_test.py` | The whole test suite (python + `psycopg2` + `bson`) |
| `META.json` | PGXN distribution metadata |
| `t1.sql`, `README.md`, `LICENSE` | |

The working directory contains many **untracked** scratch files (`perf*.py`, `ff*.py`, `pt*.py`,
`pgbson.c.OK`, `Makefile.20240117`, `MyMake.mk`, `NOTES`, built `.o`/`.so`/`.dylib`). These are the
author's local experiments and stale backups — not source of truth, and not to be modified or
cleaned up unless asked. `MyMake.mk` is a local variant of `Makefile` with Homebrew paths filled in.

## Build, install, test

The only external dependency is libbson (`brew install mongo-c-driver`, or `libbson-devel` on RH).
**Before the first build you must edit `BSON_INCLUDES` and `BSON_SHLIB` at the top of `Makefile`** to
point at the local libbson headers and shared lib; the committed values (`$(HOME)/projects/bson/...`)
work on nobody else's machine. Everything below that line is stock PGXS.

```bash
make PGUSER=postgres              # -> pgbson.o, pgbson.so (or .dylib on OS X)
make PGUSER=postgres install      # copies .so + .sql + .control into the postgres tree
# then RECONNECT (see below) before the new code is picked up
```

The author builds with the local Homebrew variant instead: `make -f MyMake.mk PGUSER=postgres`
and `make -f MyMake.mk PGUSER=postgres install`.

**Reconnect, not restart.** Postgres is process-per-connection and a backend `dlopen()`s the
shlib on the first call into it, then holds that code for the life of the process — it never
`dlclose()`s. So a *new connection* gets the newly installed `.so`; no server restart is
needed, which is why `pgbson_test.py` picks up changes on its own. What does *not* reload is
an already-open session: a `psql` left running across a `make install` keeps executing the old
C code silently, as would a pooler handing back a recycled backend. Restarting the server is
just the sledgehammer that guarantees no backend survives.

Note the `drop extension` / `create extension` in the test suite's `init()` is *not* what
reloads the C code, though it is easy to believe it is. It does re-read `pgbson--2.1.sql` from
disk, so **SQL-level** changes — a new `CREATE FUNCTION`, a changed operator binding — really
do take effect from the drop/create alone. Only the C half is tied to process lifetime.

Verify the installed shlib can actually find libbson at runtime — `ldd`/`otool -L` on the installed
`pgbson.so` must resolve `libbson-1.0.so.0`. A build that links fine but can't resolve at runtime is
the most common failure mode. On RHEL 9 the PGXS `.bc` LLVM target may fail for lack of `clang`; the
`.bc` is not needed for the extension to work.

```bash
python3 pgbson_test.py            # runs the full suite
```

The suite needs `psycopg2` and `pymongo` (for the `bson` module only). It connects using the **DSN
hardcoded at `pgbson_test.py:14`** — edit it for your local server. `init()` drops and recreates the
`pgbson` extension and a `bsontest` table on every run, so it always exercises the freshly installed
`.so`.

### Running a single test

`main()` holds a list of dicts, each keyed by a single-character marker (`pgbson_test.py:660`):

- `'-'` — run it (normal state)
- `'S'` — solo. If **any** test is marked `S`, only `S` tests run and the rest are silently skipped.
- `'M'` — mute. Skipped but reported as `skip`.

So to iterate on one test, flip its `'-'` to `'S'`, and flip it back when adding it to the suite.
Failures are reported by a test returning a non-`None` message string (or raising); returning `None`
means pass.

## Architecture

### The type is raw BSON bytes in a varlena

There is no in-memory struct. A `bson` datum *is* the BSON byte sequence with a varlena header, so
`bson`→`bytea` is a free cast (`WITHOUT FUNCTION`) and client drivers get the exact bytes back.
`storage = extended` means large values TOAST.

Consequences that pervade the C code:

- `BSON_GETARG_BSON(n)` detoasts *packed*, so all access must use `VARDATA_ANY` / `VARSIZE_ANY_EXHDR`
  (wrapped as `BSON_VARDATA_ANY`), never plain `VARDATA`.
- Every function puts a `bson_t` **on the stack** and points it at the datum with `bson_init_static`
  (macro `BSON_STATIC_INIT`) — no copying, no allocation. Anything libbson returns from an iterator

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buzzm/postgresbson](https://github.com/buzzm/postgresbson) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
