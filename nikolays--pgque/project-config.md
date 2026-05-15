---
trigger: always_on
description: PgQue (pgque) -- PgQ Universal Edition. Zero-bloat PostgreSQL queue repackaged
---

# CLAUDE.md -- PgQue

## Project

PgQue (pgque) -- PgQ Universal Edition. Zero-bloat PostgreSQL queue repackaged
from PgQ for managed database environments. Repo: NikolayS/pgque.

## Naming Convention

- `pgque` -- lowercase for schema, package names, CLI, repo, function prefixes
- `PgQue` -- capitalized form for prose, headings, README titles
- Mirrors PostgreSQL ecosystem convention (pgmq, pg_cron, psycopg)

## Architecture

Two-layer design:

- **pgque-core:** Productization of PgQ (rename, PG14+ modernization, pg_cron,
  security hardening, single-file install). Mechanical transformation of ~4,028
  lines of proven PL/pgSQL.
- **pgque-api:** Modern convenience layer (send/receive/ack/nack, DLQ, delayed
  delivery). New code (~1,500 lines) that must reduce cleanly to PgQ primitives.

See `blueprints/SPECx.md` for the full specification.
See `blueprints/SPEC.md` for PgQ internals reference (rotation mechanics,
snapshot isolation, batch_event_sql algorithm, dual-filter optimization).

## Stack

- Pure SQL + PL/pgSQL (no extensions, no `.control` file)
- Anti-extension design: `\i pgque.sql` to install
- Works on RDS, Aurora, Cloud SQL, AlloyDB, Supabase, Neon, Crunchy Bridge
- Optional pg_cron for automated ticker and maintenance
- PostgreSQL 14+ required (uses `pg_snapshot`, `xid8`)

## Style Rules

These rules are the source of truth for agentic engineering in this repo. Agents and AI coding tools must read this file before making changes.

### SQL Style

- Lowercase SQL keywords: `select`, `create function`, not `SELECT`, `CREATE FUNCTION`
- `snake_case` for all identifiers
- Schema-qualify all internal references: `pgque.queue`, not just `queue`
- **Root SQL keywords must be left-aligned within the statement.** Do not use
  decorative vertical indentation like `select` / `from` / `where` shifted by
  different amounts. This is non-negotiable for generated and source SQL.
- Put `and` / `or` at the start of continuation lines, aligned under the
  statement's `where` clause indentation.
- All `SECURITY DEFINER` functions MUST include
  `SET search_path = pgque, pg_catalog`
- Use `xid8` for transaction ID columns, `pg_snapshot` for snapshot columns
- Use modern PG14+ function names: `pg_current_xact_id()` not `txid_current()`

### Binary Units

- Always use binary units (KiB, MiB, GiB, TiB) for memory, storage, and data
  sizes in prose, reports, and documentation
- Exception: PostgreSQL config parameters use PG's native format
  (`shared_buffers = '2GB'`, `work_mem = '64MB'`)

### Shell Style

```bash
#!/usr/bin/env bash
set -Eeuo pipefail
```

- 2-space indent, no tabs
- Quote all variable expansions

### Documentation

- **Write for new users.** README and docs target someone arriving today, not
  someone migrating from an older state. Do not include "before X" or
  "previously did Y, now does Z" framing in the README or under `docs/`.
  Migration notes belong in release notes (`CHANGELOG.md` / GitHub releases),
  full stop.
- **No PR or issue numbers, and no concrete version tags, in the README or
  `docs/` unless it really helps the reader.** No `#123`, no `(see PR #456)`,
  no `refs #102, #106`, no `Breaking change in v0.2`, no `PgQue v0.2.0 has
  …`. Refer to behavior, not the change history or release labels that
  produced it. (External citations to other projects' issue trackers, used
  as evidence for a claim, are fine. Version tags are fine in roadmap-style
  contexts under `blueprints/` where the version IS the topic.) PR/issue
  numbers and version tags may appear in commit messages, release notes,
  `blueprints/`, and `CONTRIBUTING.md`.
- **Installation docs must be self-complete but not bloated.** Every
  install/quickstart doc covers: install command, ticker setup (or how to skip
  it), and role grants (`pgque_reader` / `pgque_writer` / `pgque_admin`,
  including the produce+consume case that needs both). One short snippet,
  not a tour.

### Git Commits

- Conventional Commits: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`
- Subject < 50 chars, present tense
- Never amend -- create new commits
- Never force-push unless explicitly confirmed

## File Organization

```
pgque/
  pgq/                 -- git submodule (github.com/pgq/pgq, pinned tag)
  build/
    transform.sh       -- mechanical rename + modernization script
  blueprints/          -- specs, design docs, brainstorms
    SPECx.md           -- pgque specification (the main spec)
    SPEC.md            -- PgQ internals reference (from pgq repo)
    PHASES.md          -- what ships in v0.1 vs experimental
  sql/                 -- source SQL files
    pgque.sql  -- single-file install (built from sources)
    pgque-unpgque.sql
  tests/               -- regression tests (sql/ + expected/)
  docs/                -- user-facing documentation (flat layout)
    README.md          -- index of the docs directory
    tutorial.md        -- hands-on walkthrough (start here for new users)
    reference.md       -- per-function reference for the default install
    examples.md        -- short patterns (fan-out, exactly-once, etc.)
    benchmarks.md      -- throughput numbers + methodology
    pgq-concepts.md    -- contributor glossary (batch, tick, rotation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NikolayS/PgQue](https://github.com/NikolayS/PgQue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
